# Sprint 2: Analyse Average Movie Ratings by Subscription Type

This query explores the client's idea by calculating the average rating for each movie and grouping the results by subscription type. It helps compare whether free and premium users rate movies differently, which could support future product or content decisions.

```sql
SELECT 
    mc.MovieID,
    mc.Movie_Title,
    u.Subscription_Status,
    COUNT(r.RatingID) AS TotalRatings,
    ROUND(AVG(r.Rating), 2) AS AvgRating
FROM movies_clean mc
JOIN ratings_clean r
    ON mc.MovieID = r.MovieID
JOIN users u
    ON u.UserID = r.UserID
GROUP BY 
    mc.MovieID,
    mc.Movie_Title,
    u.Subscription_Status
ORDER BY 
    AvgRating DESC;
```