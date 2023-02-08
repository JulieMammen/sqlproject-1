----Selecting key data
    SELECT  location,date
      continent,
      date,
      total_cases,
      new_cases,
      population,
      total_deaths
      FROM [Portfoliocovid].[dbo].[CovidDeaths$]
	  Order by 1,2
 ---Total cases vs total deaths(likelihood of dying if you contract covid)
    SELECT  location,date,
      continent,
      total_cases,
    population,
      total_deaths,Round((total_deaths/total_cases)*100,1) AS Deathrate
      FROM [Portfoliocovid].[dbo].[CovidDeaths$]
	  WHERE location like'%states%'
	  Order by 1,2
 ---Total cases vs population
	SELECT  location,date,
      continent,
      total_cases,
    population,
     (total_cases/NULLIF(population,0))*100 AS percentofpopulationinfected
FROM [Portfoliocovid].[dbo].[CovidDeaths$]
	  WHERE location like'%states%'
	  Order by 1,2
----Countries with highest infection rate
	 
  SELECT  location,
      continent,
     MAX(total_cases) as highestinfectioncount,
     Max (total_cases/NULLIF(population,0))*100 AS percentofpopulationinfected
FROM [Portfoliocovid].[dbo].[CovidDeaths$]
 --WHERE location like'%states%'
	  Group by location,total_cases,continent
	  Order by percentofpopulationinfected
	  OR
 SELECT  location,date,
       continent,
       MAX(total_cases) as highestinfectioncount,
       SUM(population) as population,
       SUM(total_cases)/NULLIF(SUM(population),0)*100 AS percentofpopulationinfected
FROM [Portfoliocovid].[dbo].[CovidDeaths$]
GROUP BY location,date, continent, total_cases
ORDER BY percentofpopulationinfected


---Continent with highest deathcount per population


SELECT continent, MAX(CAST(total_deaths as int)) AS TotalDeathCount
FROM [Portfoliocovid].[dbo].[CovidDeaths$]
WHERE continent is not null
GROUP BY continent
ORDER BY TotalDeathCount

Select location, Max(CAST(total_deaths as int )) AS TotalDeathCount
FROM [Portfoliocovid].[dbo].[CovidDeaths$]
Where continent is null
Group By location
Order By TotalDeathCount 


----Breakdown of countries with highest deathcount/population

SELECT continent, MAX(CAST(total_deaths as int)) AS TotalDeathCount
FROM [Portfoliocovid].[dbo].[CovidDeaths$]
WHERE continent is not null
GROUP BY continent
ORDER BY TotalDeathCount DESC

----deathpercentage across the world
Select location, isSUM(new_cases) as total_cases,Sum(cast(new_deaths as int)) AS total_deaths,(Sum(cast(new_deaths as int)) /SUM(new_cases))*100 AS DeathPercentage
FROM [Portfoliocovid].[dbo].[CovidDeaths$]
WHERE continent is not null
Group by date,location
Order by location
---Looking at the second table
Select*
From Portfoliocovid..Covidvaccinations$

---Looking at total population vs vaccination
Select *
From Portfoliocovid..CovidDeaths$ dea
Join Portfoliocovid..Covidvaccinations$ vac
ON dea.location=vac.location
and dea.date=vac.date

---looking at total population vs vaccination 
Select dea.continent,dea.date,dea.population, vac.new_vaccinations,SUM(CONVERT(int,vac.new_vaccinations)) OVER(Partition by dea.location Order by dea.location,dea.Date) AS rollingpeoplevaccinated
From Portfoliocovid..CovidDeaths$ dea
Join Portfoliocovid..Covidvaccinations$ vac
ON dea.location=vac.location
and dea.date=vac.date
WHERE dea.continent is not null
Order by 2,3
