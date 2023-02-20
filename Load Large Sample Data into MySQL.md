# Load Large Sample Data into MySQL

[StackOverflow Data Dump](https://archive.org/details/stackexchange) provides a list of free data to download.



## 1. Download Data Files

1. Choose a relatively small dataset and copy it's URL.

   * For example, askubuntu.com.7z dataset https://archive.org/download/stackexchange/askubuntu.com.7z

2. Download the file using `wget`. 

   ```bash
   mkdir sample-ubuntu-data && cd sample-ubuntu-data
   wget https://archive.org/download/stackexchange/askubuntu.com.7z
   ```

3. Unzip the 7z file.

   ```bash
   sudo apt-get install p7zip-full
   7z x askubuntu.com.7z
   ```

   

## 2. Load Data into MySQL

1. Save following SQL to a file `create_tables.sql`. It creates a database and relevant tables.

   ```SQL
   create database stackoverflow DEFAULT CHARACTER SET utf8mb4 DEFAULT COLLATE utf8mb4_bin;
   
   use stackoverflow;
   
   create table badges (
     Id INT NOT NULL PRIMARY KEY,
     UserId INT,
     Name VARCHAR(50),
     Date DATETIME
   );
   
   CREATE TABLE comments (
       Id INT NOT NULL PRIMARY KEY,
       PostId INT NOT NULL,
       Score INT NOT NULL DEFAULT 0,
       Text TEXT,
       CreationDate DATETIME,
       UserId INT NOT NULL
   );
   
   CREATE TABLE post_history (
       Id INT NOT NULL PRIMARY KEY,
       PostHistoryTypeId SMALLINT NOT NULL,
       PostId INT NOT NULL,
       RevisionGUID VARCHAR(36),
       CreationDate DATETIME,
       UserId INT NOT NULL,
       Text TEXT
   );
   CREATE TABLE post_links (
     Id INT NOT NULL PRIMARY KEY,
     CreationDate DATETIME DEFAULT NULL,
     PostId INT NOT NULL,
     RelatedPostId INT NOT NULL,
     LinkTypeId INT DEFAULT NULL
   );
   
   
   CREATE TABLE posts (
       Id INT NOT NULL PRIMARY KEY,
       PostTypeId SMALLINT,
       AcceptedAnswerId INT,
       ParentId INT,
       Score INT NULL,
       ViewCount INT NULL,
       Body text NULL,
       OwnerUserId INT NOT NULL,
       OwnerDisplayName varchar(256),
       LastEditorUserId INT,
       LastEditDate DATETIME,
       LastActivityDate DATETIME,
       Title varchar(256) NOT NULL,
       Tags VARCHAR(256),
       AnswerCount INT NOT NULL DEFAULT 0,
       CommentCount INT NOT NULL DEFAULT 0,
       FavoriteCount INT NOT NULL DEFAULT 0,
       CreationDate DATETIME
   );
   
   CREATE TABLE tags (
     Id INT NOT NULL PRIMARY KEY,
     TagName VARCHAR(50) CHARACTER SET latin1 DEFAULT NULL,
     Count INT DEFAULT NULL,
     ExcerptPostId INT DEFAULT NULL,
     WikiPostId INT DEFAULT NULL
   );
   
   
   CREATE TABLE users (
       Id INT NOT NULL PRIMARY KEY,
       Reputation INT NOT NULL,
       CreationDate DATETIME,
       DisplayName VARCHAR(50) NULL,
       LastAccessDate  DATETIME,
       Views INT DEFAULT 0,
       WebsiteUrl VARCHAR(256) NULL,
       Location VARCHAR(256) NULL,
       AboutMe TEXT NULL,
       Age INT,
       UpVotes INT,
       DownVotes INT,
       EmailHash VARCHAR(32)
   );
   
   CREATE TABLE votes (
       Id INT NOT NULL PRIMARY KEY,
       PostId INT NOT NULL,
       VoteTypeId SMALLINT,
       CreationDate DATETIME
   );
   ```

2. Connect to MySQL server and run the SQL file.

   * Relpace <HOST> and <USER> accordingly.

   ```
   mysql -h <HOST> -u <USER> -p < creates_table.sql
   ```

   Sample Command:

   ```
   mysql -h database-1.cluster-c3bottoj4h9o.ap-southeast-1.rds.amazonaws.com -u admin -p < creates_table.sql
   ```

3. Connect to MySQL server again with following command. 

   * **NOTE:** `--local-infile=1` is required to allow importing of local file.

   ```
   mysql -h <HOST> -u <USER> -p --local-infile=1
   ```

   Sample command:

   ```bash
   mysql -h database-1.cluster-c3bottoj4h9o.ap-southeast-1.rds.amazonaws.com -u admin -p --local-infile=1
   ```

4. Examine the new database and tables.

   ```sql
   use stackoverflow;
   show tables;
   ```

5. Load XML files into database table.

   ```bash
   load xml local infile 'Badges.xml' into table badges rows identified by '<row>';
   load xml local infile 'Comments.xml' into table comments rows identified by '<row>';
   load xml local infile 'PostHistory.xml' into table post_history rows identified by '<row>';
   LOAD XML LOCAL INFILE 'PostLinks.xml' INTO TABLE post_links ROWS IDENTIFIED BY '<row>';
   load xml local infile 'Posts.xml' into table posts rows identified by '<row>';
   LOAD XML LOCAL INFILE 'Tags.xml' INTO TABLE tags ROWS IDENTIFIED BY '<row>';
   load xml local infile 'Users.xml' into table users rows identified by '<row>';
   load xml local infile 'Votes.xml' into table votes rows identified by '<row>';
   ```

6. Examine the amount of data in the tables.

   ```sql
   SELECT
        table_schema as `Database`,
        table_name AS `Table`,
        table_rows AS "Rows",
        round(((data_length + index_length) / 1024 / 1024/ 1024), 2) `Size in GB`
   FROM information_schema.TABLES
   WHERE table_schema = 'stackoverflow'
   ORDER BY (data_length + index_length) DESC;  
   ```

   