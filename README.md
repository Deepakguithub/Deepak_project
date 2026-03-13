COVID-19 Data Exploration using SQL
1. Project Overview

This project focuses on exploring global COVID-19 data using SQL to analyze infection rates, death rates, and vaccination progress across different countries and continents.

The dataset was analyzed using Microsoft SQL Server, and the goal was to extract meaningful insights that could later be used for data visualization tools such as Power BI or Tableau.

Key analysis areas include:

Infection rate across countries

Death percentage among infected individuals

Population infection percentage

Global COVID-19 statistics

Vaccination progress by country

2. Tools & Technologies

SQL Server

SQL (T-SQL)

COVID-19 Dataset (CovidDeaths & CovidVaccinations tables)

3. Skills Demonstrated

This project demonstrates several important SQL skills:

Data Filtering

Aggregate Functions

Joins

Window Functions

Common Table Expressions (CTE)

Temporary Tables

Data Type Conversion

Creating Views for Data Visualization

4. Dataset Description
Table 1: CovidDeaths

Contains global COVID-19 statistics such as:

Location (Country)

Continent

Date

Population

Total Cases

New Cases

Total Deaths

New Deaths

Table 2: CovidVaccinations

Contains vaccination data including:

Location

Date

New Vaccinations

Total Vaccinations

Both tables are joined using location and date.

5. Data Exploration & Analysis
5.1 Selecting Relevant Data

The first step is filtering useful columns and excluding aggregated continent rows.

SELECT Location, Date, Total_Cases, New_Cases, Total_Deaths, Population
FROM PortfolioProject..CovidDeaths
WHERE Continent IS NOT NULL
ORDER BY Location, Date;

This provides the base dataset for further analysis.

5.2 Total Cases vs Total Deaths

This analysis calculates the death percentage among confirmed cases.

SELECT 
    Location,
    Date,
    Total_Cases,
    Total_Deaths,
    (Total_Deaths / Total_Cases) * 100 AS DeathPercentage
FROM PortfolioProject..CovidDeaths
WHERE Continent IS NOT NULL
ORDER BY Location, Date;

Insight:
This shows the likelihood of dying after contracting COVID-19 in each country.

5.3 Total Cases vs Population

This query identifies what percentage of a country's population has been infected.

SELECT 
    Location,
    Date,
    Population,
    Total_Cases,
    (Total_Cases / Population) * 100 AS PercentPopulationInfected
FROM PortfolioProject..CovidDeaths
ORDER BY Location, Date;

Insight:
Helps measure how widely COVID-19 spread relative to the population.

5.4 Countries with Highest Infection Rate

This identifies countries with the highest infection percentage.

SELECT 
    Location,
    Population,
    MAX(Total_Cases) AS HighestInfectionCount,
    MAX((Total_Cases / Population)) * 100 AS PercentPopulationInfected
FROM PortfolioProject..CovidDeaths
GROUP BY Location, Population
ORDER BY PercentPopulationInfected DESC;

Insight:
Shows which countries experienced the largest spread relative to their population.

5.5 Countries with Highest Death Count

This query finds countries with the highest number of deaths.

SELECT 
    Location,
    MAX(CAST(Total_Deaths AS INT)) AS TotalDeathCount
FROM PortfolioProject..CovidDeaths
WHERE Continent IS NOT NULL
GROUP BY Location
ORDER BY TotalDeathCount DESC;

Insight:
Highlights the countries most impacted in terms of fatalities.

5.6 Death Count by Continent
SELECT 
    Continent,
    MAX(CAST(Total_Deaths AS INT)) AS TotalDeathCount
FROM PortfolioProject..CovidDeaths
WHERE Continent IS NOT NULL
GROUP BY Continent
ORDER BY TotalDeathCount DESC;

Insight:
Compares the total deaths across continents.

5.7 Global COVID-19 Statistics
SELECT 
    SUM(New_Cases) AS TotalCases,
    SUM(CAST(New_Deaths AS INT)) AS TotalDeaths,
    SUM(CAST(New_Deaths AS INT)) / SUM(New_Cases) * 100 AS DeathPercentage
FROM PortfolioProject..CovidDeaths
WHERE Continent IS NOT NULL;

Insight:
Provides overall global statistics of infection and death rates.

5.8 Population vs Vaccination Analysis

This query calculates the cumulative number of vaccinated individuals per country using a window function.

SELECT 
    dea.Continent,
    dea.Location,
    dea.Date,
    dea.Population,
    vac.New_Vaccinations,
    SUM(CONVERT(INT, vac.New_Vaccinations)) 
        OVER (PARTITION BY dea.Location ORDER BY dea.Date) 
        AS RollingPeopleVaccinated
FROM PortfolioProject..CovidDeaths dea
JOIN PortfolioProject..CovidVaccinations vac
    ON dea.Location = vac.Location
    AND dea.Date = vac.Date
WHERE dea.Continent IS NOT NULL
ORDER BY dea.Location, dea.Date;

Insight:
Tracks vaccination progress over time.

5.9 Using CTE for Vaccination Percentage
WITH PopvsVac AS
(
SELECT 
    dea.Continent,
    dea.Location,
    dea.Date,
    dea.Population,
    vac.New_Vaccinations,
    SUM(CONVERT(INT, vac.New_Vaccinations)) 
        OVER (PARTITION BY dea.Location ORDER BY dea.Date)
        AS RollingPeopleVaccinated
FROM PortfolioProject..CovidDeaths dea
JOIN PortfolioProject..CovidVaccinations vac
    ON dea.Location = vac.Location
    AND dea.Date = vac.Date
WHERE dea.Continent IS NOT NULL
)

SELECT *,
(RollingPeopleVaccinated / Population) * 100 AS VaccinationPercentage
FROM PopvsVac;

Insight:
Calculates the percentage of population vaccinated.

5.10 Using Temporary Tables

Temporary tables were used to store intermediate calculations.

DROP TABLE IF EXISTS #PercentPopulationVaccinated;

CREATE TABLE #PercentPopulationVaccinated
(
Continent NVARCHAR(255),
Location NVARCHAR(255),
Date DATETIME,
Population NUMERIC,
New_Vaccinations NUMERIC,
RollingPeopleVaccinated NUMERIC
);

Data is inserted and later used to calculate vaccination percentages.

5.11 Creating a View for Visualization

To simplify visualization in BI tools, a view was created.

CREATE VIEW PercentPopulationVaccinated AS
SELECT 
    dea.Continent,
    dea.Location,
    dea.Date,
    dea.Population,
    vac.New_Vaccinations,
    SUM(CONVERT(INT, vac.New_Vaccinations)) 
        OVER (PARTITION BY dea.Location ORDER BY dea.Date)
        AS RollingPeopleVaccinated
FROM PortfolioProject..CovidDeaths dea
JOIN PortfolioProject..CovidVaccinations vac
    ON dea.Location = vac.Location
    AND dea.Date = vac.Date
WHERE dea.Continent IS NOT NULL;

This view can be directly connected to Power BI or Tableau dashboards.

6. Key Insights

Some countries experienced very high infection rates compared to population size.

Death percentages varied significantly between countries.

Vaccination rollout showed large differences between developed and developing regions.

Global death percentage provides an overall view of pandemic severity.

7. Conclusion

This SQL project demonstrates how data exploration techniques can uncover meaningful insights from large datasets. By using SQL features such as joins, window functions, CTEs, temporary tables, and views, the analysis provides a structured understanding of the global COVID-19 situation.

The dataset can further be used to build interactive dashboards and predictive analysis models.
