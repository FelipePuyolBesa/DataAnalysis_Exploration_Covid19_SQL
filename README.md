# DataAnalysis_Exploration_Covid19_SQL

Data exploration with SQL to a Covid 19 dataset

Here you can find the [SQL file](https://github.com/FelipePuyolBesa/DataAnalysis_Exploration_Covid19_SQL/blob/main/Covid19_DataExploration_SQL.sql)

## Objective
Make some basic data exploration on SQL to understand a given data set.

## Data set and guide
* Data taken from [https://ourworldindata.org/covid-deaths](https://ourworldindata.org/covid-deaths) about Covid 19 deaths around the globe from 01/01/2020 to 03/03/2022
* And guide made by [Alex the Analyst](https://www.youtube.com/channel/UC7cs8q-gJRlGwj4A8OmCmXg) in the video [SQL Data Exploration](https://www.youtube.com/watch?v=qfyynHBFOsM&ab_channel=AlexTheAnalyst)



## Data preparation and load
1. We created two different data sets from the original data, one that containts the deaths data and the other containts the vaccination data.

![Covid2](https://user-images.githubusercontent.com/124479181/216799458-10f69104-c7b8-4e46-bb0d-62677d3358ae.png)

3. We loaded the data set on Microsoft SQL Server in two different tables.

![Covid3](https://user-images.githubusercontent.com/124479181/216799543-0ee3eda2-2d55-408b-a612-01b7eb62d08a.png)

## SQL queries

Query all the CovidDeaths data order by location and then for date.


    SELECT * FROM  Covid19..CovidDeaths
    ORDER BY 3, 4;
    
    
!Covid4


Query  the columns we are interested

    SELECT location, date, total_cases, new_cases, total_deaths, population
    FROM Covid19..CovidDeaths
    WHERE total_cases IS NOT NULL
    ORDER BY 1, 2;
    
 !Covid5



Calculating the deaths_percentage in Chile, we have to the 2022-03-02 a total of 3.098.110 cases with a deaths percentage of 1.37%  

    SELECT location, date, total_cases, total_deaths, (total_deaths/total_cases)*100 AS deaths_percentage
    FROM Covid19..CovidDeaths
    WHERE total_cases IS NOT NULL AND location LIKE 'Chile'
    ORDER BY 1, 2

!Covid6


Chile is at Rank 90 of countries with more deaths percentage.

    SELECT location, date, total_cases, total_deaths, (total_deaths/total_cases)*100 AS deaths_percentage
    FROM Covid19..CovidDeaths
    WHERE total_cases IS NOT NULL AND date LIKE '2022-03-02'
    ORDER BY 5 DESC
    
 !Covid7

!Covid8


Almost the 16.13% of the population in Chile have had Covid 

    SELECT location, date, total_cases, population, (total_cases/population)*100 AS cases_percentage
    FROM Covid19..CovidDeaths
    WHERE location LIKE 'Chile'
    ORDER BY 1, 2;

!Covid9


The top five countries by cases percentage for the total population are countries with low population.

    SELECT TOP 5 location, date, total_cases, population, (total_cases/population)*100 AS cases_percentage
    FROM Covid19..CovidDeaths
    WHERE date LIKE '2022-03-02'
    ORDER BY 5 DESC;
    
!Covid10


Group by for looking the cases percentages

    SELECT location, population, MAX(total_cases) as HighestInfectionCount, MAX(total_cases/population)*100 as PercentagePopulationInfected
    FROM Covid19..CovidDeaths GROUP BY location, population
    ORDER BY PercentagePopulationInfected DESC
    
!Covid11


Showing the countries with highest death count by population

    SELECT location, population, MAX(total_deaths) as HighestDeathsCount, MAX(total_deaths/population)*100 as PercentagePopulationDeath
    FROM Covid19..CovidDeaths GROUP BY location, population
    ORDER BY PercentagePopulationDeath DESC

Here we can see that the top countries per percentage deaths by population are south European countries, and the highest rate is for Peru.

!Covid12


Quering the data we can see an error, the total_deaths atrribute has been define as Varchar, so we can use CAST function to used it as an integer.

    SELECT location, MAX(CAST(total_deaths AS INTEGER)) as DeathsCount
    FROM Covid19..CovidDeaths GROUP BY location
    ORDER BY DeathsCount DESC

!Covid13

Querying the data we can see an error, we have aggrupations for continents, region and income. So we have to filter the data.
All this aggrupations does not have a value in continent.

    SELECT location, MAX(CAST(total_deaths AS INTEGER)) as DeathsCount
    FROM Covid19..CovidDeaths 
    WHERE continent IS NOT NULL
    GROUP BY location
    ORDER BY DeathsCount DESC

So the top three countries by deaths are USA, Brazil and India.

!Covid14


Breaking Down the deaths count by aggrupations, we can see the total deaths on the world and  Europe, as continent, has the highest quantity.

    SELECT location, MAX(CAST(total_deaths AS INTEGER)) as DeathsCount
    FROM Covid19..CovidDeaths 
    WHERE continent IS NULL
    GROUP BY location
    ORDER BY DeathsCount DESC

!Covid15


Now we want to know the total cases by date

    SELECT date, SUM(new_cases) as Total_cases
    FROM Covid19..CovidDeaths
    WHERE continent IS NOT NULL
    GROUP BY date
    ORDER BY date

!Covid16

Now we calculate the total deaths percentage by date

    SELECT date, SUM(new_cases) as Total_cases, SUM(CAST(new_deaths AS int)) as Total_deaths, 100*SUM(CAST(new_deaths AS int))/SUM(new_cases) AS TotalDeathsPercentage
    FROM Covid19..CovidDeaths
    WHERE continent IS NOT NULL
    GROUP BY date
    ORDER BY date

!Covid17


The total deaths percentage is 1.35% to the date.

    SELECT SUM(new_cases) as Total_cases, SUM(CAST(new_deaths AS int)) as Total_deaths, 100*SUM(CAST(new_deaths AS int))/SUM(new_cases) AS TotalDeathsPercentage
    FROM Covid19..CovidDeaths
    WHERE continent IS NOT NULL

!Covid18


Join the two tables to see the new vaccinations per day

    SELECT DEA.date, DEA.continent, DEA.location,DEA.population, DEA.total_deaths, VAC.new_vaccinations FROM
    Covid19..CovidDeaths AS DEA JOIN Covid19..CovidVaccinations AS VAC
    ON DEA.date = VAC.date AND
    DEA.location = VAC.location
    WHERE DEA.continent IS NOT NULL
    ORDER BY DEA.location, DEA.date

!Covid19

Looking at the cumulative quantity new vaccinations per day in Chile


    SELECT DEA.date, DEA.continent, DEA.location,DEA.population, DEA.total_deaths, VAC.new_vaccinations, 
    SUM(CONVERT(bigint, VAC.new_vaccinations)) OVER (PARTITION BY DEA.location ORDER BY DEA.location, DEA.date) AS CumulativeVaccinations
    FROM Covid19..CovidDeaths AS DEA JOIN Covid19..CovidVaccinations AS VAC
    ON DEA.date = VAC.date AND
    DEA.location = VAC.location
    WHERE DEA.continent IS NOT NULL AND DEA.location LIKE 'Chile'
    ORDER BY DEA.location, DEA.date

!Covid20


Calculation of the accumulated percentage of vaccinations

    WITH PopvsVac (date, continent, location, population, total_deaths, vaccinations, CumulativeVaccinations)
    AS
    (
    SELECT DEA.date, DEA.continent, DEA.location,DEA.population, DEA.total_deaths, VAC.new_vaccinations, 
    SUM(CONVERT(bigint, VAC.new_vaccinations)) OVER (PARTITION BY DEA.location ORDER BY DEA.location, DEA.date) AS CumulativeVaccinations
    FROM Covid19..CovidDeaths AS DEA JOIN Covid19..CovidVaccinations AS VAC
    ON DEA.date = VAC.date AND
    DEA.location = VAC.location
    WHERE DEA.continent IS NOT NULL AND DEA.location LIKE 'Chile'
    )
    SELECT *, (CumulativeVaccinations/population)*100 AS CumulativeVaccinationsPercentage
    FROM PopvsVac

!Covid21


Calculation of the accumulated percentage of vaccinations with a temp table

    DROP TABLE IF exists #PercentPopulationVaccinated
    CREATE TABLE #PercentPopulationVaccinated

    (
    Date date,
    Continent NVARCHAR(255),
    Location NVARCHAR(255),
    Population NUMERIC,
    New_vaccinations NUMERIC,
    CumulativeVaccinations BIGINT
    )

    INSERT INTO #PercentPopulationVaccinated

    SELECT DEA.date, DEA.continent, DEA.location, DEA.population, VAC.new_vaccinations, 
    SUM(CONVERT(bigint, VAC.new_vaccinations)) OVER (PARTITION BY DEA.location ORDER BY DEA.location, DEA.date) AS CumulativeVaccinations
    FROM Covid19..CovidDeaths AS DEA JOIN Covid19..CovidVaccinations AS VAC
    ON DEA.date = VAC.date AND
    DEA.location = VAC.location
    WHERE DEA.continent IS NOT NULL AND DEA.location LIKE 'Chile'

    SELECT *, (CumulativeVaccinations/Population)*100 AS CumulativeVaccinationsPercentage
    FROM #PercentPopulationVaccinated;

!Covid22


Creating a view to be used in data visualization

    DROP VIEW IF EXISTS PercentPopulationVaccinated

    CREATE VIEW PercentPopulationVaccinated AS

        SELECT DEA.date, DEA.continent, DEA.location, DEA.population, VAC.new_vaccinations, 
        SUM(CONVERT(bigint, VAC.new_vaccinations)) OVER (PARTITION BY DEA.location ORDER BY DEA.location, DEA.date) AS CumulativeVaccinations
        FROM Covid19..CovidDeaths AS DEA JOIN Covid19..CovidVaccinations AS VAC
        ON DEA.date = VAC.date AND
        DEA.location = VAC.location
        WHERE DEA.continent IS NOT NULL AND DEA.location LIKE 'Chile'

!Covid23

## Views creation for data analysis!

    DROP VIEW IF EXISTS COVID_DB
    CREATE VIEW dbo.COVID_DB AS
        SELECT DEA.continent, DEA.location, DEA.date, DEA.population, DEA.total_cases, DEA.new_cases, DEA.total_deaths, DEA.new_deaths,
        VAC.people_vaccinated, VAC.new_vaccinations, VAC.people_fully_vaccinated
        FROM Covid19..CovidDeaths AS DEA JOIN Covid19..CovidVaccinations AS VAC
        ON DEA.date = VAC.date AND
        DEA.location = VAC.location

Querying COVID_DB view

    SELECT * FROM COVID_DB
    ORDER BY location, date

!Covid24

-- View with only countries data, cleaning aggrupations

DROP VIEW IF EXISTS COVID_DB_COUNTRIES

    CREATE VIEW COVID_DB_COUNTRIES AS
        SELECT * FROM COVID_DB
        WHERE continent IS NOT NULL

Querying COVID_DB_COUNTRIES view

    SELECT * FROM COVID_DB_COUNTRIES
    ORDER BY location, date
!Covid25


View with data for income aggrupations

    DROP VIEW IF EXISTS COVID_DB_INCOME

    CREATE VIEW COVID_DB_INCOME AS
        SELECT * FROM COVID_DB
        WHERE continent IS NULL and location LIKE '%income'

Querying COVID_DB_INCOME view

    SELECT * FROM COVID_DB_INCOME
    ORDER BY location, date

!Covid26


View with data for continents

    DROP VIEW IF EXISTS COVID_DB_CONTINENTS

    CREATE VIEW COVID_DB_CONTINENTS AS
        SELECT * FROM COVID_DB
        WHERE continent IS NULL and location IN ('Africa', 'Asia', 'Europe', 'North America', 'Oceania', 'South America')

Querying COVID_DB_CONTINENTS view

    SELECT * FROM COVID_DB_CONTINENTS
    ORDER BY location, date

!Covid27


View with data for all the world

    DROP VIEW IF EXISTS COVID_DB_WORLD

    CREATE VIEW COVID_DB_WORLD AS
        SELECT * FROM COVID_DB
        WHERE continent IS NULL and location LIKE 'WORLD'

Querying COVID_DB_WORLD view

    SELECT * FROM COVID_DB_WORLD
    ORDER BY location, date

!Covid28


### Some analysis by country

Top five countries by people fully vaccinated

    SELECT TOP 5 location, MAX(CAST(people_fully_vaccinated AS INT)) AS Fully_vaccinated
    FROM COVID_DB_COUNTRIES
    GROUP BY location
    ORDER BY Fully_vaccinated DESC

!Covid29


Top five countries by less people fully vaccinated NOT NULL

    SELECT TOP 5 location, MAX(CAST(people_fully_vaccinated AS INT)/population)*100 AS Percentage_Fully_vaccinated
    FROM COVID_DB_COUNTRIES
    GROUP BY location
    ORDER BY Percentage_Fully_vaccinated DESC

!Covid30


Top five countries by less people fully vaccinated over population NOT NULL

    SELECT TOP 5 location, MAX(CAST(people_fully_vaccinated AS INT)/population)*100 AS Percentage_Fully_vaccinated
    FROM COVID_DB_COUNTRIES
    GROUP BY location
    HAVING MAX(CAST(people_fully_vaccinated AS INT)/population)*100 > 0
    ORDER BY Percentage_Fully_vaccinated

!Covid31

### Some analysis by income

    SELECT location as income, SUM(CAST (new_cases AS BIGINT)) AS cases, SUM(CAST(new_deaths AS BIGINT)) AS deaths, SUM(CAST(new_vaccinations AS BIGINT)) as vaccinations
    FROM COVID_DB_INCOME
    GROUP BY location
    ORDER BY cases DESC

!Covid32
