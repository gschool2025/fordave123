# fordave123

-- STEP 1. Find the open dataset, download CSV and import into Database.
-- STEP 2. Creating the structure of tables
-- Airport
create table Air_Airport (
id_airport int identity(1,1) primary key, -- autoincrement 1 by 1
airport_name varchar(100),
airport_code_1 varchar(4),
airport_code_2 varchar(3),
country varchar(100),
city varchar(100))

-- Transport
create table Air_Transport (
id_transport int identity(1,1) primary key,
transport_type varchar(100))

-- Time
create table Air_Time (
id_time int identity(1,1) primary key,
time_period varchar(20),
[year] varchar(4),
[month] varchar(2),
[month_name] varchar(20),
[quarter] varchar(1),
[quarter_name] varchar(20))

-- Fact table
create table Air_Fact (
id_time int foreign key (id_time) references air_time(id_time),
id_airport int foreign key (id_airport) references air_airport(id_airport),
id_transport int foreign key (id_transport) references air_transport(id_transport),
measure numeric(20)) -- measure is maximum 20 digits - so from 0 to 99999999999999999999

-- STEP 3. Insert the data into the structure created in STEP 2.
-- insert into air_airport
select distinct
rep_airp as airport_name,
'' as airport_code_1,
'' as airport_code_2,
'' as country,
'' as city
from estat_avia_paoa_filtered_en

select * from air_airport

update air_airport
set airport_code_1='EPGD',airport_code_2='GDN',country='Poland',city='Gdansk'
where airport_name like 'GDANSK%'

-- insert into air_transport
select distinct
tra_cov as transport_type
from estat_avia_paoa_filtered_en

-- insert into air_time
select distinct
time_period,
left(time_period,4) as [year],
cast(right(time_period,2) as int) as [month],
datename(MONTH,time_period+'-01') as [month_name], -- datename should have argument like 2024-11-01
case
when cast(right(time_period,2) as int) <=3 then '1'
when cast(right(time_period,2) as int) <=6 then '2'
when cast(right(time_period,2) as int) <=9 then '3'
else '4' end as [quarter],
case
when cast(right(time_period,2) as int)<=3 then '1st quarter'
when cast(right(time_period,2) as int)<=6 then '2nd quarter'
when cast(right(time_period,2) as int)<=9 then '3rd quarter'
else '4th quarter' end as [quarter_name]
from estat_avia_paoa_filtered_en
order by 1

select * from air_time

delete from estat_avia_paoa_filtered_en where tra_cov like '%GUADEL%'

-- select the data from source table corresponding to the dimension tables
--insert into air_fact
select id_time, id_airport, id_transport, OBS_VALUE as measure
from estat_avia_paoa_filtered_en s,
Air_Airport a,
Air_Time t,
Air_Transport tr
where s.TIME_PERIOD=t.time_period
and a.airport_name=s.rep_airp
and tr.transport_type=s.tra_cov

SELECT *
  FROM [DW_S4220].[dbo].[estat_avia_paoa_filtered_en]
