

# EXO

```mysql
CREATE DATABASE testsql;
USE testsql;

CREATE TABLE foyer (
	ID_FOYER int not null,
	CLI_ID varchar(20) not null
);


load data local infile "/Users/Jr/Desktop/FOYER.csv" into table foyer fields terminated by "," lines terminated by "\r\n";

https://stackoverflow.com/questions/59993844/error-loading-local-data-is-disabled-this-must-be-enabled-on-both-the-client


#把CLI_ID BIGINT	NOT NULL 改成了 VARCHAR(20)
#把OFFER VARCHAR(10) 改成了50
CREATE TABLE client(
  CLI_ID VARCHAR(20)	NOT NULL,
  OFFRE VARCHAR(50)	NOT NULL,
  ROLE_CLI CHAR(1)	NOT NULL,
  CIVILITE VARCHAR(10),
  PRENOM VARCHAR(20),
  NOM VARCHAR(20),
  DAT_NAISS DATETIME default null,
  NUM_RUE VARCHAR(50),
  RUE VARCHAR(50),
  CMPL_ADR VARCHAR(50),
  COD_POSTL VARCHAR(5),
  VILLE VARCHAR(50),
  DAT_ACT_CLI DATETIME,
  CTU_GUID VARCHAR(20)
);

load data local infile "/Users/Jr/Desktop/CLIENT.csv" into table CLIENT fields terminated by "," lines terminated by "\r\n";




alter table FOYER add constraint FK_ID foreign key(CLI_ID) REFERENCES CLIENT(CLI_ID);
```





完全OK的

```mysql
😅#1.sortir la liste de tous les clients prénommés "JEAN" et nommés "VEU"
？自己理解题意了吗

UPDATE CLIENT
SET PRENOM = 'VEU'
WHERE PRENOM ='JEAN';




✅ #2.sortir une liste de 100 clients
SELECT *
FROM CLIENT
LIMIT 100;


✅#3.Calculer le nombre de clients titulaire
SELECT COUNT(*)
FROM CLIENT
WHERE ROLE_CLI = 'T';



#4.Calculer le nombre de clients titulaire ayant une date de naissance inconnue (valeur NULL)
🤔日期转换有问题，明明设置了DAT_NAISS DATETIME default null但是好像还是受到了(null)的影响

SELECT COUNT(*)
FROM CLIENT
WHERE ROLE_CLI = 'T'
AND DAT_NAISS IS NULL;



❌问题是mysql把所有的日期都转成了0000-00-00 00:00:00，所以我只能写成
SELECT COUNT(*)
FROM CLIENT
WHERE ROLE_CLI = 'T'
AND DAT_NAISS = '0000-00-00 00:00:00'; #为什么等号不行呢


✅#5.Calculer le nombre de clients par role client et offre
🙋？？OFFRE	VARCHAR(10) 很明显装不下INTERNET MOBILE GP和INTERNET MOBILE PRO =>导致他们都写成了INTERNET M，给后面的统计造成了错误#20
所以我自己主动将字符改成了50

SELECT OFFRE, ROLE_CLI, COUNT(*) AS COUNT
FROM CLIENT
GROUP BY OFFRE, ROLE_CLI
ORDER BY OFFRE;


没有修改前的错误版本：错误在INTERNET M上面
+------------+----------+-------+
| OFFRE      | ROLE_CLI | COUNT |
+------------+----------+-------+
| FORFAIT BL | T        |     7 |
| FORFAIT BL | U        |     9 |
| FORFAIT GP | T        |    24 |
| FORFAIT GP | U        |    38 |
| FORFAIT PR | U        |     2 |
| INTERNET G | T        |     5 |
| INTERNET M | T        |     2 |
| INTERNET M | U        |     3 |
| PREPAYE    | T        |    10 |
| PREPAYE    | U        |    10 |
+------------+----------+-------+

修改后的版本
+---------------------+----------+-------+
| OFFRE               | ROLE_CLI | COUNT |
+---------------------+----------+-------+
| FORFAIT BLOQUE      | T        |     7 |
| FORFAIT BLOQUE      | U        |     9 |
| FORFAIT GP          | T        |    24 |
| FORFAIT GP          | U        |    38 |
| FORFAIT PRO         | U        |     2 |
| INTERNET GP         | T        |     5 |
| INTERNET MOBILE GP  | T        |     1 |
| INTERNET MOBILE GP  | U        |     3 |
| INTERNET MOBILE PRO | T        |     1 |
| PREPAYE             | T        |    10 |
| PREPAYE             | U        |    10 |
+---------------------+----------+-------+




#6.Comptage du nombre de  CTU 
??? difficulte 2

SELECT COUNT(DISTINCT CTU_GUID) AS NB_CTU #DISTINCT 必要吗？
FROM CLIENT
WHERE CTU_GUID <> '(null)'; #CTU_GUID IS NULL

+--------+
| NB_CTU |
+--------+
|     99 |
+--------+



✅#7.vérifier que chaque identifiant client est bien une clef (c-a-d unique) 
??? 返回什么信息？


✅这个是最好的，数据为空，显示提示信息
  SELECT IF(CLI_ID = 0, 'No doublon in CLI_ID', CLI_ID) AS CLI_ID_DOUBLONS
  FROM CLIENT
  GROUP BY CLI_ID
  HAVING COUNT(*)>1;
  
+----------------------+
| CLI_ID_DOUBLONS      |
+----------------------+
| No doublon in CLI_ID |
+----------------------+




  ✅这个其次，数据为空，就是空
  
  SELECT CLI_ID AS DUPLICATE_CLI_ID
  FROM CLIENT
  GROUP BY CLI_ID
  HAVING COUNT(*)>1;

+------------------+
| DUPLICATE_CLI_ID |
+------------------+
| PU5561190        |
| PU5624470        |
+------------------+

+------------------+
| DUPLICATE_CLI_ID |
+------------------+
|                  |
+------------------+


🙋？？为什么无法显示提示信息呢？空与null的区别是？

	SELECT IFNULL(CLI_ID, 'all CLI_ID are not unique') AS CLI_ID_DOUBLONS
  FROM CLIENT
  GROUP BY CLI_ID
  HAVING COUNT(*)>1;
  
+-----------------+
| CLI_ID_DOUBLONS |
+-----------------+
|                 |
+-----------------+



✅#8.Comptage des clients titulaires par année et mois d'activation (fonction MONTH(Datetime) et YEAR(datetime))

值得再学习下https://juejin.im/post/5db1245fe51d452a3c6c9ffe




SELECT CONCAT(MONTH(DAT_ACT_CLI),'-', YEAR(DAT_ACT_CLI)) AS `TIME`,
			 COUNT(*) AS SUBSCRIBED_CLIENT
FROM CLIENT
WHERE ROLE_CLI ='T'
GROUP BY `TIME`;

+---------+-------------------+
| TIME    | SUBSCRIBED_CLIENT |
+---------+-------------------+
| 8-2009  |                 1 |
| 4-2010  |                 1 |
| 5-2009  |                 1 |
| 10-2010 |                 2 |
| ....... |                 . |
+---------+-------------------+



#用DATE_FORMAT也可以啊
SELECT DATE_FORMAT(DAT_ACT_CLI, '%M-%Y') AS `TIME`,
			 COUNT(*) AS SUBSCRIBED_CLIENT
FROM CLIENT
WHERE ROLE_CLI ='T'
GROUP BY `TIME`;

+--------------+-------------------+
| TIME         | SUBSCRIBED_CLIENT |
+--------------+-------------------+
| August-2009  |                 1 |
| April-2010   |                 1 |
| May-2009     |                 1 |
| October-2010 |                 2 |
| ..........   |                 . |
+--------------+-------------------+




🤮✅#9.Données d'histogramme des CTU par taille du CTU en titulaires  (nombre de compte titulaire dans un CTU): Retourner un tableau (Taille du CTU, Nombre de CTU de cette taille)

🙋？？？如何处理为null的值？
我在这里直接忽略了

✅
SELECT TEMP.CTU_SIZE, COUNT(CTU_GUID) AS NR_CTU_SIZE
FROM
  (SELECT CTU_GUID, COUNT(CTU_GUID) AS CTU_SIZE 
  FROM CLIENT
  WHERE ROLE_CLI = 'T'
  GROUP BY CTU_GUID
  HAVING CTU_GUID != '(null)') AS TEMP #HAVING CTU_GUID IS NOT NULL
GROUP BY TEMP.CTU_SIZE;

+----------+-------------+
| CTU_SIZE | NR_CTU_SIZE |
+----------+-------------+
|        1 |          37 |
|        3 |           1 |
|        2 |           4 |
+----------+-------------+


✅
🙋但是我的表格的Null值有问题！是(null)！！！下面的这个行得通哦

SELECT TEMP.CTU_SIZE, COUNT(CTU_GUID) AS NR_CTU_SIZE
FROM
  (SELECT CTU_GUID, COUNT(CTU_GUID) AS CTU_SIZE 
  FROM CLIENT
  WHERE ROLE_CLI = 'T'
  GROUP BY CTU_GUID
  HAVING CTU_GUID != '(null)') AS TEMP #HAVING CTU_GUID IS NOT NULL
GROUP BY TEMP.CTU_SIZE;




？？？为什么这种方法不行呢

SELECT TEMP.CTU_SIZE, COUNT(CTU_GUID) AS NR_CTU_SIZE
FROM
  (SELECT CTU_GUID, COUNT(CTU_GUID) AS CTU_SIZE #照理来说，COUNT(*)才把null值计算进来啊
  FROM CLIENT
  WHERE ROLE_CLI = 'T'
  GROUP BY CTU_GUID) AS TEMP
GROUP BY TEMP.CTU_SIZE;

+----------+-------------+
| CTU_SIZE | NR_CTU_SIZE |
+----------+-------------+
|        1 |          37 |
|        3 |           1 |
|        2 |           4 |
+----------+-------------+


❌
SELECT TEMP.CTU_SIZE, COUNT(CTU) AS NR_CTU_SIZE
FROM
  (SELECT IFNULL(CTU_GUID, 'EMPTY') AS CTU, COUNT(CTU_GUID) AS CTU_SIZE 
  FROM CLIENT
  GROUP BY IFNULL(CTU_GUID, 'EMPTY')) AS TEMP
GROUP BY TEMP.CTU_SIZE;



❌
下面的是计算上NULL的值的，不要用

SELECT TEMP.CTU_SIZE, COUNT(CTU_GUID) AS NR_CTU_SIZE
FROM
  (SELECT CTU_GUID, COUNT(*) AS CTU_SIZE
  FROM CLIENT
  GROUP BY CTU_GUID) AS TEMP
GROUP BY TEMP.CTU_SIZE;

+----------+-------------+
| CTU_SIZE | NR_CTU_SIZE |
+----------+-------------+
|        1 |          95 |
|        2 |           5 |
+----------+-------------+








✅#10	Calculer combien de foyers distincts existent		
SELECT COUNT(DISTINCT ID_FOYER)
FROM FOYER;




✅#11 vérifier qu'il n'y a pas de doublons dans la table FOYER	
🙋???下面的这种形式可以吗


SELECT *, COUNT(*) DUPLICATE_NB
FROM FOYER
GROUP BY ID_FOYER, CLI_ID
HAVING COUNT(*) > 1;

+----------+-----------+--------------+
| ID_FOYER | CLI_ID    | DUPLICATE_NB |
+----------+-----------+--------------+
|       21 | PU5624470 |            2 |
+----------+-----------+--------------+





✅#13.En considérant qu'un client sans foyer est célibataire, sortir la liste des clients célibataires

SELECT *
FROM CLIENT
WHERE CLI_ID NOT IN
  (SELECT CLI_ID 
  FROM FOYER);



✅#14.vérifier que les foyers ne référencent pas de clients inconnus dans la table CLIENT	

SELECT DISTINCT CLI_ID AS 'CLI_ID_NOT_IN_TAB_CLIENT' 
FROM FOYER
WHERE CLI_ID NOT IN
	(SELECT CLI_ID
  FROM CLIENT);


+--------------------------+
| CLI_ID_NOT_IN_TAB_CLIENT |
+--------------------------+
| F10031263819             |
+--------------------------+




✅# 15.Sortir la liste des identifiants foyers ayant des titulaires dont le nom commence par "MART"	

SELECT DISTINCT ID_FOYER
FROM FOYER
WHERE CLI_ID IN
  (SELECT CLI_ID
  FROM CLIENT
  WHERE NOM LIKE 'MART%'
  AND ROLE_CLI = 'T');
   
  
+----------+
| ID_FOYER |
+----------+
|      896 |
+----------+
  



✅#16.vérifier qu'un client n'est que dans un seul foyer
🙋???究竟怎么个形式确认？

SELECT CLI_ID, COUNT(*) AS IN_HOW_MANY_FOYER
FROM FOYER
GROUP BY CLI_ID
HAVING COUNT(*) >1;

+-----------+-------------------+
| CLI_ID    | IN_HOW_MANY_FOYER |
+-----------+-------------------+
| PU5624470 |                 3 |
+-----------+-------------------+



SELECT TEMP.CLI_ID EXIST_IN_SERVAL_FOYER
FROM
  (SELECT CLI_ID, COUNT(*) AS COUNT
  FROM FOYER
  GROUP BY CLI_ID) TEMP
WHERE TEMP.COUNT > 1;


+-----------------------+
| EXIST_IN_SERVAL_FOYER |
+-----------------------+
| PU5624470             |
+-----------------------+




✅#17sortir la liste des foyers ayant au moins un client anonyme ( = dont le NOM n'est qu'une suite de 19 chiffre )	
🙋??? 只显示id_foyer吗？
🙋与join比较起来谁更给力呢performance

SELECT DISTINCT ID_FOYER
FROM FOYER
WHERE CLI_ID IN
  (SELECT CLI_ID
  FROM CLIENT
  WHERE NOM REGEXP '^\\d{19}$'); #只有19个字符
  
  
+----------+
| ID_FOYER |
+----------+
|      896 |
+----------




✅#18.compter les clients par Sexe (valeurs possibles: 'M', 'F', 'Inconnu')		

 🙋???Mademoisel 容错机制？？
 🙋和23题完全一样啊
 
 SELECT SEX, COUNT(*) AS COUNT
 FROM
  (SELECT CIVILITE, 
        CASE UPPER(CIVILITE)
          WHEN 'M' THEN 'M'
          WHEN 'M.' THEN 'M'
          WHEN 'MONSIEUR' THEN 'M'
          WHEN 'MLLE' THEN 'F'
          WHEN 'MLE' THEN 'F'
          WHEN 'MME' THEN 'F'
          WHEN 'MADEMOISELlE' THEN 'F'
   				WHEN 'MADEMOISEl' THEN 'F' 
          WHEN 'MADAME' THEN 'F'
          ELSE 'INCONNU'
        END AS SEX
  FROM CLIENT) AS TEMP
GROUP BY SEX;

+---------+-------+
| SEX     | COUNT |
+---------+-------+
| M       |    26 |
| F       |    40 |
| INCONNU |    44 |
+---------+-------+



#22.Retourner pour chaque ville, le dernier client créé dans la table CLIENT

这里不能用cod_postl代替ville来做group by，因为不同的cod_postl会属于同一个城市啊


✅
#max(date)取的是最近的时间
SELECT CLI_ID, VILLE, DAT_ACT_CLI
FROM CLIENT 
WHERE (VILLE, DAT_ACT_CLI) IN
  (SELECT VILLE, MAX(DAT_ACT_CLI) AS DATERANK 
  FROM CLIENT
  WHERE VILLE != '(null)' #AND VILLE IS NOT NULL
  GROUP BY VILLE); 

+--------------+-----------------------+---------------------+
| CLI_ID       | VILLE                 | DAT_ACT_CLI         |
+--------------+-----------------------+---------------------+
| B152194      | LE TRANGER            | 2009-08-24 09:31:37 |
| B503870      | MONTPELLIER           | 2010-04-02 16:53:14 |
| PU5624470    | NANCY                 | 2011-04-12 00:00:00 |
| F20009288888 | VILLEPINTE            | 2009-11-05 00:00:00 |
| ............ | .......               | ................... |
+--------------+-----------------------+---------------------+



✅
这里要用 DENSE_RANK而不是RANK，以免有同时注册的人啊

SELECT *
FROM
    (SELECT COD_POSTL,VILLE, DAT_ACT_CLI,
            DENSE_RANK() OVER(PARTITION BY VILLE ORDER BY DAT_ACT_CLI DESC) AS MYRANK
    FROM CLIENT) AS TEMP
WHERE TEMP.MYRANK=1
AND VILLE != '(null)'; #AND VILLE IS NOT NULL




✅#19	Qu'est ce qu'une "WINDOW FUNCTION" en SQL? A quoi cela sert-il? Pouvez vous donner un exemple de fonction et d'usage	
DENSE_RANK/RANK/ROW_NUMBER/SUM/AVG/COUNT/.... OVER(PARTITION BY .... ORDER BY ... DESC/ASC) AS ..



🤢✅#20	Calculer la répartition des CTUs en profil d'équipement.Retourner un tableau profil d'équipement, nombre de CTU de ce profil pour tous les CTUs	

为什么不起作用 SET SESSION group_concat_max_len=102400 ->是因为之前OFFRE的长度是10😅
🙋 我写得太复杂了，有没有简单方法啊

✅
SELECT PROFILE_EQUIPMENT, COUNT(PROFILE_EQUIPMENT) AS NB
FROM 
		(SELECT TEMP.CTU_GUID AS CTU,
			 GROUP_CONCAT(TEMP.MYCOUNTER) AS PROFILE_EQUIPMENT
		FROM
    		(SELECT CTU_GUID, OFFRE,
           			CONCAT_WS('*',COUNT(OFFRE),OFFRE) AS MYCOUNTER
    		FROM CLIENT
    		WHERE ROLE_CLI ='T'
    		GROUP BY CTU_GUID, OFFRE) AS TEMP
		GROUP BY TEMP.CTU_GUID) AS TEMP1
GROUP BY PROFILE_EQUIPMENT;

错误的统计：错在INTERNET M上
+------------------------+----+
| PROFILE_EQUIPMENT      | NB |
+------------------------+----+
| 1*PREPAYE              |  6 |
| 1*INTERNET G           |  5 |
| 1*FORFAIT GP           | 17 |
| 1*FORFAIT BL           |  7 |
| 1*FORFAIT GP,2*PREPAYE |  1 |
| 1*INTERNET M           |  2 |
| 2*FORFAIT GP           |  3 |
| 2*PREPAYE              |  1 |
+------------------------+----+

正确的统计
+------------------------+----+
| PROFILE_EQUIPMENT      | NB |
+------------------------+----+
| 1*PREPAYE              |  6 |
| 1*INTERNET GP          |  5 |
| 1*FORFAIT GP           | 17 |
| 1*FORFAIT BLOQUE       |  7 |
| 1*FORFAIT GP,2*PREPAYE |  1 |
| 1*INTERNET MOBILE PRO  |  1 |
| 1*INTERNET MOBILE GP   |  1 |
| 2*FORFAIT GP           |  3 |
| 2*PREPAYE              |  1 |
+------------------------+----+




下面的是步骤：是拿CTU_GUID='900000682564'的这个人做例子的，看起来简单点啊


SELECT CTU_GUID, OFFRE, 
			 CONCAT_WS('*',COUNT(OFFRE),OFFRE) AS MYCOUNTER
FROM CLIENT
WHERE ROLE_CLI ='T'
AND CTU_GUID='900000682564'
GROUP BY CTU_GUID, OFFRE;
+--------------+------------+--------------+
| CTU_GUID     | OFFRE      | MYCOUNTER    |
+--------------+------------+--------------+
| 900000682564 | PREPAYE    | 2*PREPAYE    |
| 900000682564 | FORFAIT GP | 1*FORFAIT GP |
+--------------+------------+--------------+



SELECT TEMP.CTU_GUID AS CTU,
			 GROUP_CONCAT(TEMP.MYCOUNTER) AS PROFILE_EQUIPMENT
FROM
    (SELECT CTU_GUID, OFFRE,
           CONCAT_WS('*',COUNT(OFFRE),OFFRE) AS MYCOUNTER
    FROM CLIENT
    WHERE ROLE_CLI ='T'
    AND CTU_GUID='900000682564'
    GROUP BY CTU_GUID, OFFRE) AS TEMP
GROUP BY TEMP.CTU_GUID;

+--------------+------------------------+
| CTU          | PROFILE_EQUIPMENT      |
+--------------+------------------------+
| 900000682564 | 2*PREPAYE,1*FORFAIT GP |
+--------------+------------------------+





SELECT PROFILE_EQUIPMENT, COUNT(PROFILE_EQUIPMENT) AS NB
FROM 
		(SELECT TEMP.CTU_GUID AS CTU,
			 GROUP_CONCAT(TEMP.MYCOUNTER) AS PROFILE_EQUIPMENT
		FROM
    		(SELECT CTU_GUID, OFFRE,
           			CONCAT_WS('*',COUNT(OFFRE),OFFRE) AS MYCOUNTER
    		FROM CLIENT
    		WHERE ROLE_CLI ='T'
    		AND CTU_GUID='900000682564'
    		GROUP BY CTU_GUID, OFFRE) AS TEMP
		GROUP BY TEMP.CTU_GUID) AS TEMP1
GROUP BY PROFILE_EQUIPMENT;

+------------------------+--------------------------+
| PROFILE_EQUIPMENT      | COUNT(PROFILE_EQUIPMENT) |
+------------------------+--------------------------+
| 2*PREPAYE,1*FORFAIT GP |                        1 |
+------------------------+--------------------------+






🆙 目前还不行，to be continued....

SELECT CTU_GUID, OFFRE,
			 COUNT(*) OVER(PARTITION BY CTU_GUID, OFFRE) AS COUNTER
FROM CLIENT
WHERE CTU_GUID='900000682564';

+--------------+------------+---------+
| CTU_GUID     | OFFRE      | COUNTER |
+--------------+------------+---------+
| 900000682564 | FORFAIT GP |       1 |
| 900000682564 | PREPAYE    |       2 |
| 900000682564 | PREPAYE    |       2 |
+--------------+------------+---------+




SELECT DISTINCT GROUP_CONCAT(COUNTER, OFFRE SEPARATOR '*')
FROM 
    (SELECT *,
           COUNT(*) OVER(PARTITION BY CTU_GUID, OFFRE) AS COUNTER
    FROM CLIENT
    WHERE CTU_GUID='900000682564') AS TEMP
;

+--------------------------------------------+
| GROUP_CONCAT(COUNTER, OFFRE SEPARATOR '*') |
+--------------------------------------------+
| 1FORFAIT GP*2PREPAYE*2PREPAYE              |
+--------------------------------------------+







✅#21.Question similaire sur les foyers (nombre de foyer par profil équipement du foyer)	

✅有没有简单的方法啊

SELECT PROFILE_EQUIPEMENT, COUNT(*) AS NB_FOYER
FROM
        (SELECT *, GROUP_CONCAT(RES) AS PROFILE_EQUIPEMENT
        FROM
            (SELECT *,
                   CONCAT_WS('*',COUNTER,OFFRE) AS RES
            FROM
                (SELECT ID_FOYER, OFFRE, COUNT(*) AS COUNTER
                FROM
                    (SELECT ID_FOYER, F.CLI_ID,PRENOM, OFFRE
                    FROM FOYER F
                    LEFT JOIN CLIENT C
                    ON F.CLI_ID = C.CLI_ID
                    WHERE ROLE_CLI ='T'
                    AND OFFRE IS NOT NULL
                    ORDER BY ID_FOYER) AS TEMP
                GROUP BY ID_FOYER, OFFRE
                ORDER BY ID_FOYER) AS TEMP1) AS TEMP2
        GROUP BY ID_FOYER) AS TEMP3
GROUP BY PROFILE_EQUIPEMENT
;



具体步骤
1/

SELECT F.ID_FOYER, F.CLI_ID,PRENOM, OFFRE
FROM FOYER F
LEFT JOIN CLIENT C
ON F.CLI_ID = C.CLI_ID
ORDER BY F.ID_FOYER;
+----------+--------------+-------------+----------------+
| ID_FOYER | CLI_ID       | PRENOM      | OFFRE          |
+----------+--------------+-------------+----------------+
|       21 | F10030732567 | LILIANE     | FORFAIT GP     |
|       21 | F20031610536 | AHMED       | FORFAIT GP     |
|       21 | P204278038   | CHRYSTEL    | FORFAIT BLOQUE |
|       21 | F20035566700 | AZZA        | FORFAIT GP     |
|       21 | F20035137099 | GUILLAUME   | FORFAIT GP     |
|      896 | F10031263819 | NULL        | NULL           |
|      896 | B836487      | FLORIAN     | INTERNET GP    |
|      896 | F20009298305 | PHILIPPE    | FORFAIT GP     |
|      896 | F20009298305 | PHILIPPE222 | FORFAIT GP     |
|      896 | F20009298305 | PHILIPPE333 | FORFAIT GP     |
|      896 | PU4917607    | KAMEL       | PREPAYE        |
|      896 | PU5624470    | LAURA       | PREPAYE        |
|      896 | PU5624470    | LAURA222    | PREPAYE        |
|      896 | PU5624470    | LAURA333    | PREPAYE        |
|    85966 | F10030264334 | ROSE        | FORFAIT GP     |
|    85966 | PU4847155    | CHRISTINE   | PREPAYE        |
+----------+--------------+-------------+----------------+


2/
SELECT ID_FOYER, OFFRE, COUNT(*)
FROM
    (SELECT ID_FOYER, F.CLI_ID,PRENOM, OFFRE
    FROM FOYER F
    LEFT JOIN CLIENT C
    ON F.CLI_ID = C.CLI_ID
    WHERE OFFRE IS NOT NULL
    ORDER BY ID_FOYER) AS TEMP
GROUP BY ID_FOYER, OFFRE
ORDER BY ID_FOYER
;

+----------+----------------+----------+
| ID_FOYER | OFFRE          | COUNT(*) |
+----------+----------------+----------+
|       21 | FORFAIT BLOQUE |        1 |
|       21 | FORFAIT GP     |        4 |
|      896 | FORFAIT GP     |        3 |
|      896 | INTERNET GP    |        1 |
|      896 | PREPAYE        |        4 |
|    85966 | FORFAIT GP     |        1 |
|    85966 | PREPAYE        |        1 |
+----------+----------------+----------+


3/
SELECT *,
			 CONCAT_WS('*',COUNTER,OFFRE) 
FROM
    (SELECT ID_FOYER, OFFRE, COUNT(*) AS COUNTER
    FROM
        (SELECT ID_FOYER, F.CLI_ID,PRENOM, OFFRE
        FROM FOYER F
        LEFT JOIN CLIENT C
        ON F.CLI_ID = C.CLI_ID
        WHERE OFFRE IS NOT NULL
        ORDER BY ID_FOYER) AS TEMP
    GROUP BY ID_FOYER, OFFRE
    ORDER BY ID_FOYER) AS TEMP1
    ;
+----------+----------------+---------+------------------------------+
| ID_FOYER | OFFRE          | COUNTER | CONCAT_WS('*',COUNTER,OFFRE) |
+----------+----------------+---------+------------------------------+
|       21 | FORFAIT BLOQUE |       1 | 1*FORFAIT BLOQUE             |
|       21 | FORFAIT GP     |       4 | 4*FORFAIT GP                 |
|      896 | FORFAIT GP     |       3 | 3*FORFAIT GP                 |
|      896 | INTERNET GP    |       1 | 1*INTERNET GP                |
|      896 | PREPAYE        |       4 | 4*PREPAYE                    |
|    85966 | FORFAIT GP     |       1 | 1*FORFAIT GP                 |
|    85966 | PREPAYE        |       1 | 1*PREPAYE                    |
+----------+----------------+---------+------------------------------+


4/

SELECT *, GROUP_CONCAT(RES) AS PROFILE_EQUIPEMENT
FROM
    (SELECT *,
           CONCAT_WS('*',COUNTER,OFFRE) AS RES
    FROM
        (SELECT ID_FOYER, OFFRE, COUNT(*) AS COUNTER
        FROM
            (SELECT ID_FOYER, F.CLI_ID,PRENOM, OFFRE
            FROM FOYER F
            LEFT JOIN CLIENT C
            ON F.CLI_ID = C.CLI_ID
            WHERE OFFRE IS NOT NULL
            ORDER BY ID_FOYER) AS TEMP
        GROUP BY ID_FOYER, OFFRE
        ORDER BY ID_FOYER) AS TEMP1) AS TEMP2
GROUP BY ID_FOYER
        ;
+----------+----------------+---------+------------------+--------------------------------------+
| ID_FOYER | OFFRE          | COUNTER | RES              | PROFILE_EQUIPEMENT                   |
+----------+----------------+---------+------------------+--------------------------------------+
|       21 | FORFAIT BLOQUE |       1 | 1*FORFAIT BLOQUE | 1*FORFAIT BLOQUE,4*FORFAIT GP        |
|      896 | FORFAIT GP     |       3 | 3*FORFAIT GP     | 3*FORFAIT GP,1*INTERNET GP,4*PREPAYE |
|    85966 | FORFAIT GP     |       1 | 1*FORFAIT GP     | 1*FORFAIT GP,1*PREPAYE               |
+----------+----------------+---------+------------------+--------------------------------------+


5/
SELECT PROFILE_EQUIPEMENT, COUNT(*) AS NB_FOYER
FROM
        (SELECT *, GROUP_CONCAT(RES) AS PROFILE_EQUIPEMENT
        FROM
            (SELECT *,
                   CONCAT_WS('*',COUNTER,OFFRE) AS RES
            FROM
                (SELECT ID_FOYER, OFFRE, COUNT(*) AS COUNTER
                FROM
                    (SELECT ID_FOYER, F.CLI_ID,PRENOM, OFFRE
                    FROM FOYER F
                    LEFT JOIN CLIENT C
                    ON F.CLI_ID = C.CLI_ID
                    WHERE OFFRE IS NOT NULL
                    ORDER BY ID_FOYER) AS TEMP
                GROUP BY ID_FOYER, OFFRE
                ORDER BY ID_FOYER) AS TEMP1) AS TEMP2
        GROUP BY ID_FOYER) AS TEMP3
GROUP BY PROFILE_EQUIPEMENT
;

+--------------------------------------+----+
| PROFILE_EQUIPEMENT                   | NB |
+--------------------------------------+----+
| 1*FORFAIT BLOQUE,4*FORFAIT GP        |  1 |
| 3*FORFAIT GP,1*INTERNET GP,4*PREPAYE |  1 |
| 1*FORFAIT GP,1*PREPAYE               |  1 |
+--------------------------------------+----+




✅#23.Compter le nombre de clients par sexe ('M','F','Inconnu') sur la base de la civilité	
la même réponse que #18
```

































还没做的

```mysql
																						
#12compter les clients de la table CLIENT par type de foyer, en incluant ceux qui ne sont pas en foyer
```









问题集锦

```mysql
#22.Retourner pour chaque ville, le dernier client créé dans la table CLIENT

???解除了这个mode之后有什么风险吗

SELECT DAT_ACT_CLI, COUNT(COD_POSTL)
FROM CLIENT
GROUP BY COD_POSTL
;
ERROR 1055 (42000): Expression #1 of SELECT list is not in GROUP BY clause and contains nonaggregated column 'testsql.CLIENT.DAT_ACT_CLI' which is not functionally dependent on columns in GROUP BY clause; this is incompatible with sql_mode=only_full_group_by


```

