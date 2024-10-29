# netflix_sql_project

       select * from nextflix;


# 1. Count the number of Movies vs TV Shows

        select 
	type,count(*) total_count
 	from nextflix
	group by 1
	order by 2 desc


	
# 2. Find the most common rating for movies and TV shows
        
	with cte as (
	select type,rating ,count(*) total_count,
	rank()over(partition by type order by count(*) desc) rnk
	from nextflix
	group by 1,2
	order by 3 desc)

	select type, rating
	from cte
	where rnk=1
	
# 3. List all movies released in a specific year (e.g., 2020)
        
	select * from nextflix 
	where type = 'Movie' and release_year = 2020
	
# 4. Find the top 5 countries with the most content on Netflix

	select
	unnest(string_to_array(country, ',') ),
	count(*) total_content
	from nextflix
	group by 1
	order by 2 desc  
	limit 5

	
# 5. Identify the longest movie
        
	select * from nextflix
	where duration = ( select max(duration) from nextflix)
	and  type = 'Movie'


# 6. Find content added in the last 5 years
        
	select * from nextflix 
	where date_added>= current_date - Interval '5 year'    
	
# 7. Find all the movies/TV shows by director 'Rajiv Chilaka'!

	select *  from nextflix    
	where director  ilike  '%Rajiv Chilaka%'  

	
# 8. List all TV shows with more than 5 seasons

	select * from nextflix 
	where type = 'TV Show'  and split_part(duration,' ', 1)::numeric > 5    


	
# 9. Count the number of content items in each genre

	select unnest(string_to_array(listed_in,',')) as genre , count(show_id) total_number
	from nextflix
	group by 1


# 10.Find each year and the average numbers of content release in India on netflix. Return top 5 year with highest avg content release!

	select 
	extract(year from date_added) as year , 
	count(show_id),
        round(count(show_id)::numeric /(select count(*) from nextflix where country = 'India')::numeric *100,2) as content_avg    
	from nextflix  
	where country= 'India'
	group by 1

	
# 11. List all movies that are documentaries
	select * from 
	nextflix 
	where type = 'Movie' and listed_in = 'Documentaries'

	
# 12. Find all content without a director

	select * from nextflix
	where director is null


	
# 13. Find how many movies actor 'Salman Khan' appeared in last 10 years!

        select * from nextflix
	where type = 'Movie' and casts like '%Salman Khan%' 
	and release_year > extract(year from  current_date) - 10
	
# 14. Find the top 10 actors who have appeared in the highest number of movies produced in India.

	select 
	unnest(string_to_array(casts,',')) as casts,
	count(*) total_count from nextflix
	where country like '%India%' and type = 'Movie'
	group by 1
	order by 2 desc
	limit 10


	
# 15.Categorize the content based on the presence of the keywords 'kill' and 'violence' in the description field. Label content containing these keywords as 'Bad' and all other content as 'Good'. Count how many items fall into each category.


	select 
        case 
	when  description ilike '%kill%' or description ilike  '%violence%' then 'Bad' 
        else 'Good'
        end category,
	count(show_id)
        from nextflix
        group by category
