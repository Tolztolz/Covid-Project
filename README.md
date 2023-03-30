# Covid-Project
A data analysis of COVID-19 trends using SQL and Power BI

"Data Analysis Process"
The Covid data sets was gotten from https://ourworldindata.org/coronavirus.  The dataset contains Datas dated from February 2020 till April 2021
Excel was used for a bit of cleaning and SQL was largely used in cleaning, query and exploration of the data.
The skills used in querying the data include Joins, CTE's, Temp Tables, Windows Functions, Aggregate Functions, Creating Views, Converting Data Types.
Power BI was then used for the visualization.

```sql
/*
Covid 19 Data Cleaning and Exploration 
*/

![covid proj power bi](https://user-images.githubusercontent.com/114151260/228989251-97075162-033a-4417-b0eb-5a319f18e5aa.png)

![Power BI dashboard screenshot](![covid proj power bi](https://user-images.githubusercontent.com/114151260/228989037-26110603-019b-4344-90a0-ce9dea42ace8.png).png)

select *
from PortfolioProj..CovidDeaths
where continent is not null
order by 3, 4

select *
from PortfolioProj..CovidVaccinations
order by 3, 4

-- selecting the datas needed

select location, date, total_cases, new_cases, total_deaths, population
from PortfolioProj..CovidDeaths
where continent is not null
order by 1, 2

-- looking at total cases per death

select location, date, total_cases, total_deaths, (total_deaths/total_cases)*100 as Deathpercase
from PortfolioProj..CovidDeaths
 where continent is not null
order by 1, 2

select location, date, total_cases, total_deaths, (total_deaths/total_cases)*100 as Deathpercase
from PortfolioProj..CovidDeaths
where location like '%Nigeria%'
-- where continent is not null
order by 1, 2

select location, date, total_cases, total_deaths, (total_deaths/total_cases)*100 as Deathpercase
from PortfolioProj..CovidDeaths
where location like '%States%'
-- where continent is not null
order by 1, 2

-- looking at total cases vs population

select location, date, total_cases, total_deaths, population, (total_cases/population)*100 as CasePerPopulation
from PortfolioProj..CovidDeaths
--where location like '%States%'
where continent is not null
order by 1, 2

--looking at countries with highest infection rate compared population

select location, population, max(total_cases) as HighestInfectionCount, max(total_cases/population)*100 as PercentagePopulationInfected
from PortfolioProj..CovidDeaths
----where location like '%States%'
 where continent is not null
group by location, population
order by PercentagePopulationInfected desc

-- looking at continent with highest infection rate
select continent, max(cast(total_cases as int)) as TotalInfectionsCount
from PortfolioProj..CovidDeaths
----where location like '%States%'
 where continent is not null
group by continent
order by TotalInfectionsCount desc

-- country with highest death rate per population

select location, max(cast(total_deaths as int)) as TotalDeathCount
from PortfolioProj..CovidDeaths
--where location like '%States%'
where continent is not null
group by location
order by TotalDeathCount desc

-- lets do by continent

select continent, max(cast(total_deaths as int)) as TotalDeathCount
from PortfolioProj..CovidDeaths
--where location like '%States%'
where continent is not null
group by continent
order by TotalDeathCount desc

select location, max(cast(total_deaths as int)) as TotalDeathCount
from PortfolioProj..CovidDeaths
--where location like '%States%'
where continent is null
group by location
order by TotalDeathCount desc

-- showing the continent with highest death count per population
select continent, max(cast(total_deaths as int)) as TotalDeathCount
from PortfolioProj..CovidDeaths
--where location like '%States%'
where continent is not null
group by continent
order by TotalDeathCount desc

select continent, max(cast(total_deaths as int)) as TotalDeathCount
from PortfolioProj..CovidDeaths
--where location like '%States%'
where continent is not null
group by continent
order by TotalDeathCount desc

-- Global numbers

select date, sum(new_cases) as total_cases, sum(cast (new_deaths as int)) as total_deaths, (sum(cast (new_deaths as int))/sum(new_cases))*100 as DeathPercentage
from PortfolioProj..CovidDeaths
where continent is not null
group by date
order by 1,2

select  sum(new_cases) as total_cases, sum(cast (new_deaths as int)) as total_deaths, (sum(cast (new_deaths as int))/sum(new_cases))*100 as DeathPercentage
from PortfolioProj..CovidDeaths
where continent is not null
--group by date
order by 1,2


-- now on vaccinations

select *
from CovidDeaths as Dea
join CovidVaccinations as Vac
	on  dea.location = Vac.location
	and dea.date = vac.date
order by 1,2

---looking at total population vs vaccinations

select dea.continent, dea.location, dea.date, dea.population, vac.new_vaccinations, sum(cast(vac.new_vaccinations as int)) over (Partition by dea.location)
from CovidDeaths as Dea
join CovidVaccinations as Vac
	on  dea.location = Vac.location
	and dea.date = vac.date
where dea.continent is not null
order by 2, 3 

-- we can use convert instead of cast to change the data type

select dea.continent, dea.location, dea.date, dea.population, vac.new_vaccinations, sum(convert (int, vac.new_vaccinations)) over (Partition by dea.location order by dea.location, dea.date) as PeopleVaccinations
from CovidDeaths as Dea
join CovidVaccinations as Vac
	on  dea.location = Vac.location
	and dea.date = vac.date
where dea.continent is not null
order by 2, 3 

-- using cte to make further calculations as we cant use value for a column derived directly

With PopvsVac (continent, location, date, population, New_vaccinations, PeopleVaccinated)
as
(
select dea.continent, dea.location, dea.date, dea.population, vac.new_vaccinations, sum(convert (int, vac.new_vaccinations)) over (Partition by dea.location order by dea.location, dea.date) as PeopleVaccinated
from CovidDeaths as Dea
join CovidVaccinations as Vac
	on  dea.location = Vac.location
	and dea.date = vac.date
where dea.continent is not null
--order by 2, 3 
)
select *, (PeopleVaccinated/Population)*100
from PopvsVac

--- using temp table instead of cte

drop table if exists #PercentPopulationVaccinated
create table #PercentPopulationVaccinated
(continent nvarchar(255), 
location nvarchar(255), 
date datetime,
population numeric,
New_vaccinations numeric, 
PeopleVaccinated numeric
)

insert into #PercentPopulationVaccinated
select dea.continent, dea.location, dea.date, dea.population, vac.new_vaccinations, sum(convert (int, vac.new_vaccinations)) over (Partition by dea.location order by dea.location, dea.date) as PeopleVaccinated
from CovidDeaths as Dea
join CovidVaccinations as Vac
	on  dea.location = Vac.location
	and dea.date = vac.date
--where dea.continent is not null
--order by 2, 3 

select *, (PeopleVaccinated/Population)*100
from #PercentPopulationVaccinated


---- creating views for later visualizations
-- View 1
Create View PercentPopulationVaccinated as
select dea.continent, dea.location, dea.date, dea.population, vac.new_vaccinations, sum(convert (int, vac.new_vaccinations)) over (Partition by dea.location order by dea.location, dea.date) as PeopleVaccinated
from CovidDeaths as Dea
join CovidVaccinations as Vac
	on  dea.location = Vac.location
	and dea.date = vac.date
where dea.continent is not null
--order by 2, 3

select *
From PercentPopulationVaccinated

--View 2
Create View worldstats as
select  sum(new_cases) as total_cases, sum(cast (new_deaths as int)) as total_deaths, (sum(cast (new_deaths as int))/sum(new_cases))*100 as DeathPercentage
from PortfolioProj..CovidDeaths
where continent is not null
--group by date
--order by 1,2

select *
From worldstats

-- View 3
Create view DeathCountPerContinent as
select continent, max(cast(total_deaths as int)) as TotalDeathCount
from PortfolioProj..CovidDeaths
--where location like '%States%'
where continent is not null
group by continent
--order by TotalDeathCount desc

select *
From DeathCountPerContinent

--View 4
Create View DeathCountPerCountry as
select location, max(cast(total_deaths as int)) as TotalDeathCount
from PortfolioProj..CovidDeaths
--where location like '%States%'
where continent is not null
group by location
--order by TotalDeathCount desc

select *
From DeathCountPerCountry
order by TotalDeathCount desc

--View 5
Create View InfectionRatePerCountry as
select location, population, max(total_cases) as HighestInfectionCount, max(total_cases/population)*100 as PercentagePopulationInfected
from PortfolioProj..CovidDeaths
----where location like '%States%'
 where continent is not null
group by location, population
--order by PercentagePopulationInfected desc

select *
From InfectionRatePerCountry
order by HighestInfectionCount desc

--- view 6

Create View InfectionRatePerContinent as
select continent, max(cast(total_cases as int)) as TotalInfectionsCount
from PortfolioProj..CovidDeaths
----where location like '%States%'
 where continent is not null
group by continent
--order by TotalInfectionsCount desc

select *
From InfectionRatePerContinent
order by TotalInfectionsCount desc



