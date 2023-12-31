# Covid_data_analysis
covid_19_data_exploration(SQL)
#metadata 
It is SQL based Codes which are used to take insights from the covid_death data taken from the website of "Our world in data"
Authentication of data: the data is hosted by oxford universities scientists.
the data is between from 03-01-2020 to 02-08-2023.(dMY)
data retrieved from this link: https://ourworldindata.org/explorers/coronavirus-data-explorer

SQL codes used for the exploration of data.

--totalcases vs total death worldwide
SELECT  sum(total_deaths) AS total_deaths_worldwide , sum(total_cases) AS total_cases_worldwide
 FROM `coherent-span-381814.covid.Covid_deaths` 




--total per% of worldwide deaths using sub query

SELECT
  total_deaths_worldwide,
  total_cases_worldwide,
  (total_deaths_worldwide / total_cases_worldwide)*100 AS case_fatality_rate
FROM (
  SELECT
    SUM(total_deaths) AS total_deaths_worldwide,
    SUM(total_cases) AS total_cases_worldwide
  FROM `coherent-span-381814.covid.Covid_deaths`
);



-- no. of people dye in pakistan due to covid_19
SELECT
  total_deaths_worldwide,
  total_cases_worldwide,
  (total_deaths_worldwide / total_cases_worldwide)*100 AS case_fatality_rate
FROM (
  SELECT
    SUM(total_deaths) AS total_deaths_worldwide,
    SUM(total_cases) AS total_cases_worldwide
  FROM `coherent-span-381814.covid.Covid_deaths`
  where location = 'Pakistan'
);



--Total cases vs worldwide population
SELECT total_cases_worldwide,
       total_population_of_world,
       (total_cases_worldwide/total_population_of_world)*100
       FROM
          (Select   sum(Population) AS total_population_of_world,  sum(total_cases) AS total_cases_worldwide  
          From `coherent-span-381814.covid.Covid_deaths`
            );


-- Countries with Highest Infection Rate compared to Population

Select Location, Population, MAX(total_cases) as HighestInfectionCount,  Max((total_cases/population))*100 as PercentPopulationInfected
From `coherent-span-381814.covid.Covid_deaths`
--Where location like '%states%'
Group by Location, Population
order by PercentPopulationInfected desc


-- Countries with Highest Death Count per Population

Select Location, MAX(Total_deaths) as TotalDeathCount
From `coherent-span-381814.covid.Covid_deaths`
GROUP BY location
order by TotalDeathCount desc

- Showing contintents with the highest death count per population

Select continent, MAX(cast(Total_deaths as int)) as TotalDeathCount
From `coherent-span-381814.covid.Covid_deaths`
--Where location like '%states%'
Where continent is not null 
Group by continent
order by TotalDeathCount desc




-- Total Population vs Vaccinations
-- Shows Percentage of Population that has recieved at least one Covid Vaccine

Select dea.continent, dea.location, dea.date, dea.population, vac.new_vaccinations,
SUM(vac.new_vaccinations) OVER (Partition by dea.Location Order by dea.location, dea.Date) as RollingPeopleVaccinated
--, (RollingPeopleVaccinated/population)*100
From `coherent-span-381814.covid.Covid_deaths` dea
Join `coherent-span-381814.covid.covid_vaccination` vac
	On dea.location = vac.location
	and dea.date = vac.date
where dea.continent is not null 
order by 2,3



--CTE

WITH PopvsVac 
AS
(
  Select dea.continent, dea.location, dea.date, dea.population, vac.new_vaccinations,
SUM(vac.new_vaccinations) OVER (Partition by dea.Location Order by dea.location, dea.Date) as RollingPeopleVaccinated
From `coherent-span-381814.covid.Covid_deaths` dea
Join `coherent-span-381814.covid.covid_vaccination` vac
	On dea.location = vac.location
	and dea.date = vac.date
where dea.continent is not null 
)
Select *, (RollingPeopleVaccinated/Population)*100 
From PopvsVac
