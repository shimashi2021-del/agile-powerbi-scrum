# MySQL Database Setup Guide for StreamFlix

## 1. Set up the database connection

```sql
CREATE DATABASE IF NOT EXISTS DataLensStreaming;
USE DataLensStreaming;
```

## 2. Creating genres table

```sql
CREATE TABLE genres (
  GenreID INT PRIMARY KEY,
  Genre_Name VARCHAR(100) NOT NULL
);
```

## 3. Creating movies_clean table

```sql
CREATE TABLE movies_clean (
  MovieID INT PRIMARY KEY,
  Movie_Title VARCHAR(255) NOT NULL,
  Year INT,
  Language VARCHAR(100),
  Country VARCHAR(100),
  Total_Views INT
);
```

## 4. Creating movie_genres table

```sql
CREATE TABLE movie_genres (
  MovieID INT NOT NULL,
  GenreID INT NOT NULL,
  PRIMARY KEY (MovieID, GenreID),
  CONSTRAINT fk_movie_genres_movie
    FOREIGN KEY (MovieID)
    REFERENCES movies_clean(MovieID),
  CONSTRAINT fk_movie_genres_genre
    FOREIGN KEY (GenreID)
    REFERENCES genres(GenreID)
);
```

## 5. Creating users table

```sql
CREATE TABLE users (
  UserID INT PRIMARY KEY,
  Age INT,
  Gender VARCHAR(20),
  Country VARCHAR(50),
  Subscription_Status VARCHAR(20),
  Total_Watch_Time INT,
  Device VARCHAR(50)
);
```

## 6. Creating ratings table

```sql
CREATE TABLE ratings (
  RatingID INT PRIMARY KEY,
  UserID INT NOT NULL,
  MovieID INT NOT NULL,
  Rating DECIMAL(2,1),
  Rating_Timestamp DATETIME,
  CONSTRAINT fk_ratings_user
    FOREIGN KEY (UserID)
    REFERENCES users(UserID),
  CONSTRAINT fk_ratings_movie
    FOREIGN KEY (MovieID)
    REFERENCES movies_clean(MovieID)
);
```

## 7. Data Import Example

```sql
SHOW VARIABLES LIKE 'secure_file_priv';
```

```sql
LOAD DATA INFILE 'C:/ProgramData/MySQL/MySQL Server 8.0/Uploads/movies_clean.csv'
INTO TABLE movies_clean
FIELDS TERMINATED BY ','
ENCLOSED BY '"'
LINES TERMINATED BY '\n'
IGNORE 1 ROWS
(MovieID, Movie_Title, Year, Language, Country, Total_Views);
```

## 8. Validate relationships

### A. Count validation

#### I. Count genres

```sql
SELECT COUNT(*) FROM genres;
```

#### II. Count movies

```sql
SELECT COUNT(*) AS movie_count FROM movies_clean;
```

#### III. Count movie genres

```sql
SELECT COUNT(*) FROM movie_genres;
```

#### IV. Count ratings

```sql
SELECT COUNT(*) AS rating_count FROM ratings;
```

#### V. Count users

```sql
SELECT COUNT(*) AS user_count FROM users;
```

### B. Validate Users -> Ratings relationship

```sql
SELECT
    r.RatingID,
    r.UserID,
    u.Age,
    u.Gender,
    u.Country,
    u.SubscriptionStatus,
    u.TotalWatchTime,
    u.Device,
    r.MovieID,
    r.Rating,
    r.Timestamp
FROM ratings r
JOIN users u
    ON r.UserID = u.UserID
LIMIT 20;
```

### C. Validate Movies -> Ratings relationship

```sql
SELECT
    r.RatingID,
    r.MovieID,
    m.Movie_Title,
    m.Year,
    m.Language,
    m.Country,
    m.Total_Views,
    r.UserID,
    r.Rating,
    r.Timestamp
FROM ratings r
JOIN movies_clean m
    ON r.MovieID = m.MovieID
LIMIT 20;
```

### D. Validate Users + Movies + Ratings together

```sql
SELECT
    r.RatingID,
    u.UserID,
    u.Age,
    u.Gender,
    u.Country AS UserCountry,
    u.SubscriptionStatus,
    m.MovieID,
    m.Movie_Title,
    m.Country AS MovieCountry,
    m.Language,
    m.Year,
    r.Rating,
    r.Timestamp
FROM ratings r
JOIN users u
    ON r.UserID = u.UserID
JOIN movies_clean m
    ON r.MovieID = m.MovieID
LIMIT 20;
```

### E. Check orphan UserID records in ratings

This checks whether any rating belongs to a user that does not exist in users.

```sql
SELECT
    r.UserID,
    COUNT(*) AS orphan_rating_count
FROM ratings r
LEFT JOIN users u
    ON r.UserID = u.UserID
WHERE u.UserID IS NULL
GROUP BY r.UserID;
```

### F. Check orphan MovieID records in ratings

This checks whether any rating belongs to a movie that does not exist in movies_clean.

```sql
SELECT
    r.MovieID,
    COUNT(*) AS orphan_rating_count
FROM ratings r
LEFT JOIN movies_clean m
    ON r.MovieID = m.MovieID
WHERE m.MovieID IS NULL
GROUP BY r.MovieID;
```

### G. Check how many users have ratings

```sql
SELECT
    COUNT(DISTINCT r.UserID) AS users_with_ratings
FROM ratings r;
```

### H. Check users without ratings

```sql
SELECT
    u.UserID,
    u.Age,
    u.Gender,
    u.Country,
    u.SubscriptionStatus,
    u.TotalWatchTime,
    u.Device
FROM users u
LEFT JOIN ratings r
    ON u.UserID = r.UserID
WHERE r.UserID IS NULL;
```

### I. Check movies without ratings

```sql
SELECT
    m.MovieID,
    m.Movie_Title,
    m.Year,
    m.Language,
    m.Country,
    m.Total_Views
FROM movies_clean m
LEFT JOIN ratings r
    ON m.MovieID = r.MovieID
WHERE r.MovieID IS NULL;
```