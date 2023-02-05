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

2. We loaded the data set on Microsoft SQL Server in two different tables.

![Covid3](https://user-images.githubusercontent.com/124479181/216799543-0ee3eda2-2d55-408b-a612-01b7eb62d08a.png)

## SQL queries

Query all the CovidDeaths data order by location and then for date.


    SELECT * FROM  Covid19..CovidDeaths
    ORDER BY 3, 4;
    
    
![Covid4](https://user-images.githubusercontent.com/124479181/216799578-b65f8ab9-8d55-433b-900b-fd47467840e6.png)

Query  the columns we are interested

    SELECT location, date, total_cases, new_cases, total_deaths, population
    FROM Covid19..CovidDeaths
    WHERE total_cases IS NOT NULL
    ORDER BY 1, 2;
    
 ![Covid5](https://user-images.githubusercontent.com/124479181/216799588-b8de273e-4534-4174-98a6-20acd98bae21.png)



Calculating the deaths_percentage in Chile, we have to the 2022-03-02 a total of 3.098.110 cases with a deaths percentage of 1.37%  

    SELECT location, date, total_cases, total_deaths, (total_deaths/total_cases)*100 AS deaths_percentage
    FROM Covid19..CovidDeaths
    WHERE total_cases IS NOT NULL AND location LIKE 'Chile'
    ORDER BY 1, 2

![Covid6](https://user-images.githubusercontent.com/124479181/216799592-f4c0b8e0-34e1-4f79-80c9-06d0acb73041.png)


Chile is at Rank 90 of countries with more deaths percentage.

    SELECT location, date, total_cases, total_deaths, (total_deaths/total_cases)*100 AS deaths_percentage
    FROM Covid19..CovidDeaths
    WHERE total_cases IS NOT NULL AND date LIKE '2022-03-02'
    ORDER BY 5 DESC
    
 ![Covid7](https://user-images.githubusercontent.com/124479181/216799598-3c1b7c99-20a4-430e-90ae-13d02ef01c87.png)

![Covid8](https://user-images.githubusercontent.com/124479181/216799610-36d361e5-08bd-427e-bcb6-efae08cfa830.png)



Almost the 16.13% of the population in Chile have had Covid 

    SELECT location, date, total_cases, population, (total_cases/population)*100 AS cases_percentage
    FROM Covid19..CovidDeaths
    WHERE location LIKE 'Chile'
    ORDER BY 1, 2;

![Covid9](https://user-images.githubusercontent.com/124479181/216799618-34be1487-a447-4b42-877c-46c82031a3a4.png)


The top five countries by cases percentage for the total population are countries with low population.

    SELECT TOP 5 location, date, total_cases, population, (total_cases/population)*100 AS cases_percentage
    FROM Covid19..CovidDeaths
    WHERE date LIKE '2022-03-02'
    ORDER BY 5 DESC;
    
![Covid10](https://user-images.githubusercontent.com/124479181/216799622-4047f4b5-a777-41b2-b0bd-f5fbd4e362a0.png)


Group by for looking the cases percentages

    SELECT location, population, MAX(total_cases) as HighestInfectionCount, MAX(total_cases/population)*100 as PercentagePopulationInfected
    FROM Covid19..CovidDeaths GROUP BY location, population
    ORDER BY PercentagePopulationInfected DESC
    
![Covid11](https://user-images.githubusercontent.com/124479181/216799633-9e276f34-5925-4335-8e06-3da2f291d328.png)


Showing the countries with highest death count by population

    SELECT location, population, MAX(total_deaths) as HighestDeathsCount, MAX(total_deaths/population)*100 as PercentagePopulationDeath
    FROM Covid19..CovidDeaths GROUP BY location, population
    ORDER BY PercentagePopulationDeath DESC

Here we can see that the top countries per percentage deaths by population are south European countries, and the highest rate is for Peru.

![Covid12](https://user-images.githubusercontent.com/124479181/216799647-aea0b607-1854-404c-b2fd-e83f05b8b54f.png)


Quering the data we can see an error, the total_deaths atrribute has been define as Varchar, so we can use CAST function to used it as an integer.

    SELECT location, MAX(CAST(total_deaths AS INTEGER)) as DeathsCount
    FROM Covid19..CovidDeaths GROUP BY location
    ORDER BY DeathsCount DESC

![Covid13](https://user-images.githubusercontent.com/124479181/216799652-4732b513-f3b2-44f2-a96d-893ab471b03b.png)

Querying the data we can see an error, we have aggrupations for continents, region and income. So we have to filter the data.
All this aggrupations does not have a value in continent.

    SELECT location, MAX(CAST(total_deaths AS INTEGER)) as DeathsCount
    FROM Covid19..CovidDeaths 
    WHERE continent IS NOT NULL
    GROUP BY location
    ORDER BY DeathsCount DESC

So the top three countries by deaths are USA, Brazil and India.

![Covid14](https://user-images.githubusercontent.com/124479181/216799657-77d216db-83d8-4ae4-8896-7973abe08078.png)


Breaking Down the deaths count by aggrupations, we can see the total deaths on the world and  Europe, as continent, has the highest quantity.

    SELECT location, MAX(CAST(total_deaths AS INTEGER)) as DeathsCount
    FROM Covid19..CovidDeaths 
    WHERE continent IS NULL
    GROUP BY location
    ORDER BY DeathsCount DESC

![Covid15](https://user-images.githubusercontent.com/124479181/216799679-a91c2d29-220d-4d96-b178-a916e813e425.png)


Now we want to know the total cases by date

    SELECT date, SUM(new_cases) as Total_cases
    FROM Covid19..CovidDeaths
    WHERE continent IS NOT NULL
    GROUP BY date
    ORDER BY date

![Covid16](https://user-images.githubusercontent.com/124479181/216799685-8eb5436c-3753-4b46-9ea5-0fae0eba573d.png)

Now we calculate the total deaths percentage by date

    SELECT date, SUM(new_cases) as Total_cases, SUM(CAST(new_deaths AS int)) as Total_deaths, 100*SUM(CAST(new_deaths AS int))/SUM(new_cases) AS TotalDeathsPercentage
    FROM Covid19..CovidDeaths
    WHERE continent IS NOT NULL
    GROUP BY date
    ORDER BY date

![Covid17](https://user-images.githubusercontent.com/124479181/216799697-ea0d8b13-de08-4dbc-8b6b-65efd4149f51.png)


The total deaths percentage is 1.35% to the date.

    SELECT SUM(new_cases) as Total_cases, SUM(CAST(new_deaths AS int)) as Total_deaths, 100*SUM(CAST(new_deaths AS int))/SUM(new_cases) AS TotalDeathsPercentage
    FROM Covid19..CovidDeaths
    WHERE continent IS NOT NULL

![Covid18](https://user-images.githubusercontent.com/124479181/216799699-931e42d6-5583-45f6-9c40-4c4b839bb1f5.png)


Join the two tables to see the new vaccinations per day

    SELECT DEA.date, DEA.continent, DEA.location,DEA.population, DEA.total_deaths, VAC.new_vaccinations FROM
    Covid19..CovidDeaths AS DEA JOIN Covid19..CovidVaccinations AS VAC
    ON DEA.date = VAC.date AND
    DEA.location = VAC.location
    WHERE DEA.continent IS NOT NULL
    ORDER BY DEA.location, DEA.date

![Covid19](https://user-images.githubusercontent.com/124479181/216799714-245f5e4c-4010-4080-99f4-9a33313ffce7.png)

Looking at the cumulative quantity new vaccinations per day in Chile


    SELECT DEA.date, DEA.continent, DEA.location,DEA.population, DEA.total_deaths, VAC.new_vaccinations, 
    SUM(CONVERT(bigint, VAC.new_vaccinations)) OVER (PARTITION BY DEA.location ORDER BY DEA.location, DEA.date) AS CumulativeVaccinations
    FROM Covid19..CovidDeaths AS DEA JOIN Covid19..CovidVaccinations AS VAC
    ON DEA.date = VAC.date AND
    DEA.location = VAC.location
    WHERE DEA.continent IS NOT NULL AND DEA.location LIKE 'Chile'
    ORDER BY DEA.location, DEA.date

![Covid20](https://user-images.githubusercontent.com/124479181/216799722-8b440036-8a1d-4af2-82a6-13d9c3315cd9.png)


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

![Covid21](https://user-images.githubusercontent.com/124479181/216799736-757e493e-a62a-4d6f-ae4a-093d9e4febdf.png)


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

![Covid22](https://user-images.githubusercontent.com/124479181/216799747-eb374916-281b-42f8-9a10-ac159b268da5.png)


Creating a view to be used in data visualization

    DROP VIEW IF EXISTS PercentPopulationVaccinated

    CREATE VIEW PercentPopulationVaccinated AS

        SELECT DEA.date, DEA.continent, DEA.location, DEA.population, VAC.new_vaccinations, 
        SUM(CONVERT(bigint, VAC.new_vaccinations)) OVER (PARTITION BY DEA.location ORDER BY DEA.location, DEA.date) AS CumulativeVaccinations
        FROM Covid19..CovidDeaths AS DEA JOIN Covid19..CovidVaccinations AS VAC
        ON DEA.date = VAC.date AND
        DEA.location = VAC.location
        WHERE DEA.continent IS NOT NULL AND DEA.location LIKE 'Chile'

![Covid23](https://user-images.githubusercontent.com/124479181/216799756-a679ff14-ffce-4f07-82bd-fb4db8eb0e8d.png)

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

![Covid24](https://user-images.githubusercontent.com/124479181/216799766-cf6c2334-6e18-43b5-959f-e0eae5e60650.png)

-- View with only countries data, cleaning aggrupations

DROP VIEW IF EXISTS COVID_DB_COUNTRIES

    CREATE VIEW COVID_DB_COUNTRIES AS
        SELECT * FROM COVID_DB
        WHERE continent IS NOT NULL

Querying COVID_DB_COUNTRIES view

    SELECT * FROM COVID_DB_COUNTRIES
    ORDER BY location, date
![Covid25](https://user-images.githubusercontent.com/124479181/216799769-540e7745-ba6e-45aa-86a4-eef42f3cdfb9.png)


View with data for income aggrupations

    DROP VIEW IF EXISTS COVID_DB_INCOME

    CREATE VIEW COVID_DB_INCOME AS
        SELECT * FROM COVID_DB
        WHERE continent IS NULL and location LIKE '%income'

Querying COVID_DB_INCOME view

    SELECT * FROM COVID_DB_INCOME
    ORDER BY location, date

![Covid26](https://user-images.githubusercontent.com/124479181/216799774-cf23c4d1-760b-43fb-85e7-817edd5c3484.png)


View with data for continents

    DROP VIEW IF EXISTS COVID_DB_CONTINENTS

    CREATE VIEW COVID_DB_CONTINENTS AS
        SELECT * FROM COVID_DB
        WHERE continent IS NULL and location IN ('Africa', 'Asia', 'Europe', 'North America', 'Oceania', 'South America')

Querying COVID_DB_CONTINENTS view

    SELECT * FROM COVID_DB_CONTINENTS
    ORDER BY location, date

![Covid27](https://user-images.githubusercontent.com/124479181/216799781-a6fdcd7d-9fdb-42aa-a323-ba06d63cba42.png)


View with data for all the world

    DROP VIEW IF EXISTS COVID_DB_WORLD

    CREATE VIEW COVID_DB_WORLD AS
        SELECT * FROM COVID_DB
        WHERE continent IS NULL and location LIKE 'WORLD'

Querying COVID_DB_WORLD view

    SELECT * FROM COVID_DB_WORLD
    ORDER BY location, date

![Covid28](https://user-images.githubusercontent.com/124479181/216799793-9db70874-cbfa-40d7-8815-e755c5480bc1.png)


### Some analysis by country

Top five countries by people fully vaccinated

    SELECT TOP 5 location, MAX(CAST(people_fully_vaccinated AS INT)) AS Fully_vaccinated
    FROM COVID_DB_COUNTRIES
    GROUP BY location
    ORDER BY Fully_vaccinated DESC

![Covid29](https://user-images.githubusercontent.com/124479181/216799802-6ed6fb84-9eb5-4c0b-a586-a696ca88a209.png)


Top five countries by less people fully vaccinated NOT NULL

    SELECT TOP 5 location, MAX(CAST(people_fully_vaccinated AS INT)/population)*100 AS Percentage_Fully_vaccinated
    FROM COVID_DB_COUNTRIES
    GROUP BY location
    ORDER BY Percentage_Fully_vaccinated DESC

![Covid30](https://user-images.githubusercontent.com/124479181/216799810-70c9e212-28d9-4768-940f-5ba2cb9a0d36.png)


Top five countries by less people fully vaccinated over population NOT NULL

    SELECT TOP 5 location, MAX(CAST(people_fully_vaccinated AS INT)/population)*100 AS Percentage_Fully_vaccinated
    FROM COVID_DB_COUNTRIES
    GROUP BY location
    HAVING MAX(CAST(people_fully_vaccinated AS INT)/population)*100 > 0
    ORDER BY Percentage_Fully_vaccinated

![Covid31](https://user-images.githubusercontent.com/124479181/216799823-14bc8e0e-4145-41cd-b229-a0418d6b27e0.png)

### Some analysis by income

    SELECT location as income, SUM(CAST (new_cases AS BIGINT)) AS cases, SUM(CAST(new_deaths AS BIGINT)) AS deaths, SUM(CAST(new_vaccinations AS BIGINT)) as vaccinations
    FROM COVID_DB_INCOME
    GROUP BY location
    ORDER BY cases DESC

![Covid32](https://user-images.githubusercontent.com/124479181/216799834-d250f321-19d7-4459-8ed7-6a010d198bae.png)
