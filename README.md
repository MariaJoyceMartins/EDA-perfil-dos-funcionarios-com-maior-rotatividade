# EDA-perfil-dos-funcionarios-com-maior-rotatividade

Identificar os principais perfis de funcionários com maior rotatividade:

# Analisar: 
Idade, gênero, região, tempo de empresa, tempo de viagem ao trabalho, cargo, salário, nível educacional, etc.
# Objetivo: 
Identificar padrões que contribuem para a rotatividade e direcionar ações estratégicas para retenção de talentos.

-- LIMPEZA DOS DADOS --
-- Alterando o nome da coluna age --
ALTER TABLE hr_data.hr CHANGE age Age int;

-- Verificando se há uma coluna NULL --
SELECT *
FROM hr_data.hr
WHERE attrition IS NULL
OR age IS NULL
OR DistanceFromHome IS NULL
OR gender IS NULL
OR JobSatisfaction IS NULL
OR WorkLifeBalance IS NULL
OR YearsSinceLastPromotion IS NULL
OR PercentSalaryHike IS NULL
OR PerformanceRating IS NULL;


-- excluindo uma coluna sem valores
ALTER TABLE hr_data.hr
DROP  total_de_ThreePerformanceRating;

SET SQL_SAFE_UPDATES=0;

-- Deletando linhas com valores NULL --
DELETE FROM hr_data.hr
WHERE attrition IS NULL
AND age IS NULL
AND DistanceFromHome IS NULL
AND JobSatisfaction IS NULL
AND WorkLifeBalance IS NULL
AND YearsSinceLastPromotion IS NULL
AND PercentSalaryHike IS NULL
AND PerformanceRating IS NULL;


 -- ANALISE DOS DADOS --
-- Descobre a idade média das pessoas que pediram demissão --
SELECT AVG(Age) AS Age, attrition
FROM hr_data.hr
WHERE attrition = 'Yes'
GROUP BY attrition;
-- Consulta retorna: 33 anos -- 

-- Comparação entre a idade média dos pediram demissão vs os que pediram -- 
SELECT AVG(Age), attrition
FROM hr_data.hr
GROUP BY attrition;

-- Consulta retorna: 33 anos para quem pediu demissão e 37 anos para quem não pediu --
-- Análise: Pessoas mais novas pedem demissão em comparação a quem não pediu, vale investigar se a empresa está sabendo lidar com pessoas nessa faixa, porém, não há uma diferença gritante entre as suas variaveis --

-- Seleciona o média de distancia da casa até o trabalho por taxa de demissão
SELECT AVG(DistanceFromHome) AS media_da_DistanceFromHome, attrition
FROM hr_data.hr
GROUP BY attrition;

-- Análise: As pessoas que pediram demissão moram em média a 10km/h do trabalho e junto a isso, moram mais longe do que as que não pediram demissão que neste caso estão em média a 8 km/h do trabalho

-- Descubre qual foi o aumento salarial de cada pessoa que pediu demissão e sua respectiva quantidade --- 
SELECT COUNT(*) AS contagem_de_PercentSalaryHike, PercentSalaryHike, attrition
FROM hr_data.hr
WHERE attrition = 'Yes'
GROUP BY attrition, PercentSalaryHike
ORDER BY contagem_de_PercentSalaryHike DESC;

-- Analise: pessoas com menores aumentos salariais pediram mais demissão, sendo elas: aumento do menor valor possivel 11% (41), 13% (34), 12% (33) contra aos que tiveram maior aumento e pediram demissão: 25%% (1), 21% (5), 24% (6)

-- Descobre a média de aumento salarial das pessoas que pediram e não pediram demissão --
SELECT AVG(PercentSalaryHike) AS media_de_PercentSalaryHike, attrition
FROM hr_data.hr
GROUP BY attrition;
-- Por outro lado, a média de aumento salarial das pessoas que pediram e não pediram demissão é a mesma, neste caso: 15% --

-- Descobre a média salarial das pessoas que pediram e não pediram demissão --
SELECT AVG(MonthlyIncome) AS media_de_MonthlyIncome, attrition
FROM hr_data.hr
GROUP BY attrition;  

-- Analise: as pessoas que pediram demissão ganham em média 2.045 a menos que as pessoas que não pediram demissão, sendo a média de quem pediu demissão: 4.787 contra 6.832 das que não pediram

-- Descobre a quantidade de homens e mulheres que pediram demissão --
SELECT COUNT(*) as Contagem_De_attrition_Por_Gender, gender, attrition
FROM hr_data.hr
WHERE attrition = 'Yes'
GROUP BY attrition, gender;


-- Seleciona o Ranking de satisfação no trabalho --
SELECT COUNT(*) As contagem_de_JobSatisfaction, JobSatisfaction, Attrition
FROM hr_data.hr
WHERE attrition = 'Yes'
GROUP BY attrition, JobSatisfaction
ORDER BY COUNT(*) DESC;

-- Ranking:
-- 1 Lugar: High
-- 2 Lugar: Low
-- 3 Lugar: Very High

-- Seleciona o percentual de participação de cada satisfacao no trabalho

SELECT
    JobSatisfaction,
    COUNT(*) AS contagem,
    100.0 * COUNT(*) / SUM(COUNT(*)) OVER() AS porcentagem
FROM
    hr_data.hr
WHERE 
    attrition = 'Yes'
GROUP BY
    JobSatisfaction;

-- Descobre a contagem da variavel 'PerformanceRating (Avaliação de Performance)'
SELECT COUNT(PerformanceRating), PerformanceRating
FROM hr_data.hr
WHERE attrition = 'Yes'
GROUP BY PerformanceRating;

-- Análise: 200 pessoas que pediram demissão tinham performance 3 (Excelente) enquanto 37 tinham nota 4 (supera as expectativas).

-- Seleciona a participação percentual dos anos desda ultima promocao em relacao aos pedidos de demissao
SELECT
    YearsSinceLastPromotion,
    COUNT(*) AS contagem,
    100.0 * COUNT(*) / SUM(COUNT(*)) OVER() AS porcentagem
FROM
    hr_data.hr
WHERE 
    attrition = 'Yes'
GROUP BY
    YearsSinceLastPromotion;

-- Calcula a participação percentual de cada categoria e cria um ranking do maior para o menor --
SELECT
  COUNT(*) AS contagem, WorkLifeBalance, Attrition, 
   100.0 * COUNT(*) / SUM(COUNT(*)) OVER() AS porcentagem
FROM 
   hr_data.hr
WHERE 
   Attrition = 'Yes'
GROUP BY 
   Attrition, WorkLifeBalance
ORDER BY porcentagem DESC;
 
-- Calcula a participação percentual de cada nota individualmente, em relação ao seu respectivo total --
SELECT
  COUNT(CASE WHEN Attrition = 'Yes' AND PerformanceRating = '3' THEN 1 END) * 100.0
  / COUNT(CASE WHEN PerformanceRating = '3' THEN 1 END) AS percentagem_attrition_yes
FROM
   hr_data.hr;

-- Como esperado, para a nota 3, tivemos 16%, ou seja, do total de notas 3 (Excellent) dadas por todos os colaboradores, 16% dos que pediram demissão tinham nota 3 de performance

SELECT
  COUNT(CASE WHEN Attrition = 'Yes' AND PerformanceRating = '4' THEN 1 END) * 100.0
  / COUNT(CASE WHEN PerformanceRating = '4' THEN 1 END) AS percentagem_attrition_yes
FROM
   hr_data.hr;
   
-- A nota 4 (Acima das expectativas) das pessoas que pediram demissão em relaçao ao total de funcionarios com nota 4 também é 16%'
