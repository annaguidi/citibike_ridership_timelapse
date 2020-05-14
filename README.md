# Citibike Ridership Timelapse
A look at how citibike ridership evolved post-coronavirus in march of 2020

## Context & Description of Project:

Somewhere between the second and third week of March 2020, the spread of covid-19 in New York City became increasingly apparent, causing, among other effects, people to stay home and work remotely, and subway ridership to decrease by up to [92%](https://brooklyneagle.com/articles/2020/04/08/new-york-city-subway-ridership-down-92-percent-due-to-coronavirus/) in an effort to social distance.

The goal of this project is to analyze the March Citibike dataset, to see if and where there has been a decrease (or perhaps increase) in ridership, and find an effective way to visualize these changes within the month.

## Dependencies:

* see requirement.txt file

## The Code:

### 1. Data QA + Preprocessing

Citibike data can be found [here](https://s3.amazonaws.com/tripdata/index.html)
I used the [March 2020 dataset](https://s3.amazonaws.com/tripdata/202003-citibike-tripdata.csv.zip).

Initially, I was not sure whether I would end up visualizing differences in ridership by hour or by day, so after transforming the starttime and stoptime of each ride to a datetimestamp using `.to_datetime`, I create 4 new columns (start_hour, start_day, end_hour, end_day).

I have used this dataset before, and know from previous QA (see my other citibike repos), that it is a high quality dataset- so while my QA in this repo is not as extensive as it could be, I still check that each citibike station has only 1 lat/long associated with it. I then create 2 dataframes- one where I sum the number of rides across each Start Station by hour and day (number_pickups), and a second dataframe where I do the same for total dropoffs (number_dropoffs). We now know the number of total pickups and dropoffs for every station for every hour and day in the month of March 2020.

However, I am interested in the total activity, so I then do an outer join on the datasets (which I QA), fill missing values with 0s, and create a new column, `activity`, which is just a sum of the `pickups` and `dropoffs` columns, as well as a `net_pickups` column, which is pickups - dropoffs. I never ended up using this last column, but might in the future.

Now for some geoprocessing:
* I make a `geometry` column by creating Points out of the station latitudes and longitudes using the `Shapely` library
* Initialize the crs
* Then change the crs to to `espg = 2263`

And with that, we are done with the data processing! In hindsight could have done more QA, but let's continue.

### 2. Initial Data Analysis

In my personal experience, it's better to do some "boring", simple data analysis and visualizations before jumping into something more detailed and mapping out the data. So I created a simple line graph showing total ridership, across ALL stations, for each day of the month of March, 2020. You can see a peak on March 9th, and then a big decline after the 14th. Something definitely happened:

![img3](https://github.com/annaguidi/citibike_ridership_timelapse/blob/master/pics/newplot%20(2).png)

Based on the above graph, I loosely defined everything pre-March 15th as before the effects of covid19 fully hit NYC. If you think about it, on [March 9th, there were still only 16 confirmed cases of coronavirus, and by March 25th, there were 17,800](https://en.wikipedia.org/wiki/COVID-19_pandemic_in_New_York_City). Somewhere in the middle is March 14th.

With that in mind, I plotted total ridership activity of the first week of the month vs. the last week of the month:

![image2](https://github.com/annaguidi/citibike_ridership_timelapse/blob/master/pics/newplot%20(3).png)

Yeah, there is a difference, but looking back at the [weather of those 2 weeks](https://www.accuweather.com/en/us/new-york/10007/march-weather/349727), I realized this comparison did not make any sense, as I was comparing cold to warm days, which must have indubiously had a big effect (sorry for the poor visualization- the icons at the bottom symbolize whether it was a cold or warm day for the 4th week of March, the icons on the top symbolize the weather for the first week):

![img3](https://github.com/annaguidi/citibike_ridership_timelapse/blob/master/days_and_weather.png)

We are clearly not comapring apples to apples, and because March is a notoriously heterogneous month in terms of weather, I just decided to write a function to compare various days to each other that matched in weekday and temperature.

2 cold (5C-11C) Thursdays with almost identical weather:

![img4](https://github.com/annaguidi/citibike_ridership_timelapse/blob/master/pics/newplot%20(4).png)

pretty big difference, especially in morning commuter bump, but the slope duing non-commute hours looks almost identical

2 warm (7C-21C) Fridays:

![img5](https://github.com/annaguidi/citibike_ridership_timelapse/blob/master/pics/newplot%20(5).png)

like thursday, the slopes look very similar. The morning commute bump is already waning on March 13th, either because it's a Friday, or because the effect of coronavirus is already affecting Citibike activity. Again, we see how morning commute spike is affected more than evening commute spike.

2 Saturdays- this time the Post-Corona Saturday is warmer than the pre-Corona one

![img6](https://github.com/annaguidi/citibike_ridership_timelapse/blob/master/pics/newplot%20(6).png)

The 21st is significantly warmer than the 14th (18C vs 14C), and yet ridership is lower, but follows similar patterns, so it's almost certain to say that due to covid-19, fewer riders are using citibikes.

Comparing 3 consecutive Wednesdays of almost identical temperature

![img7](https://github.com/annaguidi/citibike_ridership_timelapse/blob/master/pics/newplot%20(7).png)

this comparison is very insightful, because the constant of temperature is held the same, and we do see a really big drop in ridership on the third week of March. The decline is most evident during the morning commute.

So, Observations thus far:

* Weather can have a greater impact than covid during non-commute hours
* Coronavirus did have an impact on Citibike ridership:
  -  morning commute spike most impacted
  - evening bump still there, but less of a spike, more gradual increase in citibike activity culminating at 6pm
* Weekend distributions look the same, just less activity in later weeks of March

**Caveat: Obviously, these observations are based on very limited data- a good next step would be to pull in at least February data or March data from 2019 and compare post-covid ridership activity to a historical average rather than do this more empirical day to day comparison.**

### 2. Some GIS visualizations

So now we know that there are differences in citibike activity pre- and post-Covid, but we do not know *where* and at which stations these differences are playing out the most, so we will try to use some mapping to figure that out.

Using matplotlib, I plotted 2 days (one pre- and post-covid) against each other, making sure they were both on the same weekday, adn holding the hour of day constant. Here are some examples:

![img8](https://github.com/annaguidi/citibike_ridership_timelapse/blob/master/pics/mpl5122612.png)
![img9](https://github.com/annaguidi/citibike_ridership_timelapse/blob/master/pics/mpl13172717.png)
![img10](https://github.com/annaguidi/citibike_ridership_timelapse/blob/master/pics/mpl14152115.png)

Right off the bat, midtown and FiDi citbike docks seem to have the biggest decrease in activity, more so in certain hours (at first glance, difference at 5pm > difference at 3 pm).

Getting a better understanding of how ridership varies over time is kind of cumbersome this way however- I would have to loop through every hour and scroll. So there are 2 options I could think of to make this visualization more dynamic: 1. stitch images together across all 24h to create a GIF showing patterns in ridership for 2 days (one pre-corona day, one post-corona day), or use another library that allows for some animation, and just use 1 image as opposed to 2 juxtaposed images, and find a way to visualize quantified differences in ridership activity instead.

I decided to try out the second option using plotly express' scatter_mapbox, and to find the best way plot the *difference in ridership activity*. It's very easy to use and use you can use your own basemap using mapbox. There are some limitations in what you can do, but it's a great way to get started.

The days I decided to compare were 2 Wednsdays, the 5th vs the 19th, because they had almost identical temperature, and the 5th was still in the very early days of coronavirus, while by the 19th, the effects of covid19 were already impacting a significant portion of New Yorker's lives.

I did have to do a little more data work: I in order to get the difference in activity, I had to split the dataframe on the day, then merge, and then create a new column `activity_difference`, using the same steps I did outlined before in pre-processing of the data. There is probably a better way to do this, but oh well. From there I also computed what percent of activity of March 5th we observe on March 19th for each station (`percent_difference`).

In my first attempt to convey difference in ridership, I used percent_activity and pass it into the 'opacities' argument of scatter_mapbox, so that we still see a bigger circle for busy citibike docks, but if docks are seeing less traffic than usual, they are more transparent, like a shadow of their former selves (note: if there is more activity on March 19th than March 15th, you have to floor it to 1, or 100%, or it won't work).

![img11](https://github.com/annaguidi/citibike_ridership_timelapse/blob/master/pics/newplot.png)

This is not great, because when transparent markers overlap, they look more opaque, plus if activity was = 0 in the post-corona day, but not in the pre-corona day, there is no way to visualize that.

Attempt 2 consisted of experimenting with color scales instead of opacity, so again, the radius will just convey how popular a citibike dock is given the 24 hours and other docks for a given day (in our case, March 19th), but the color scale conveys how activity measures up ot pre-coronavirus March 5th. For the first attempt, I just used activity_difference as input for the color scale:

![img12](https://github.com/annaguidi/citibike_ridership_timelapse/blob/master/pics/Screen-Recording-2020-05-14-at-11.10.27-PM.gif)

The legend and color scale will completely change from day to day unless you set a `range_color=[min_val,max_val]`. This was also difficult, because I think the animation is easiest to comprehend when the midpoint is 0 and you have different colors above or below that (you can set `color_continuous_midpoint`, which is super helfpul), but the difference in activity was so skewed towards negative (far fewer rides than usual as opposed to more, which happened so rarely), AND changes from hour to hour drastically. In hindsight, I should have normalized this data column to have a mean of 0 for each hour. Food for thought!

I played around with more colorschemes:

![img13](https://github.com/annaguidi/citibike_ridership_timelapse/blob/master/pics/newplot%20(1).png)

Obviously Midtown, FiDi are most affected, and these differences phase out the further away we move out. The outlying citibike stations' activity seem barely affected at all.

Here I switch to using percents instead of number difference again, and I think it's my favourite and maybe the easiest way to comprehend the changes between ridership in the 2 days we are comparing:

![video14](https://github.com/annaguidi/citibike_ridership_timelapse/blob/master/pics/Screen-Recording-2020-05-14-at-11.11.41-PM.gif)

For my last tweak or attempt, I used discrete colors, instead of a color scale, to show whether a citibike dock was underutilized ( < 75% activity), about the same ( 75% <= activity <= 1.25%), or more activity than usual ( > 1.25%).

![img15](https://github.com/annaguidi/citibike_ridership_timelapse/blob/master/pics/newplot_final.png)

I will let you decide which of these you like best.


### 3. Conclusion, Thoughts, Improvements, and Next Steps

In addition to some of the observations I mentioned earlier, it's pretty clear that fewer people are using citibike to commute to work given the above graphs, and that the change in ridership is way more pronounced at commute peak hours (8am, 5-6pm) than during other times of the day. It would be interesting to run the animation for a weekend day as well, and see if we see any citibike dock stations jump out there as well.

This analysis could be more sound by pulling in more data and the average or median of several post-covid days to several pre-covid days. The difference in activity should have also been normalized and before that maybe visualized with a histogram.

I think there are some very interesting models that could be built to predict a ridership decline or increase score of individual docks given certain information discussed here.

Overall, my thoughts are that visualizing absence of something is very tricky. There obviously has been a big decrease in ridership, and I think even though my visualization is a good first step, there are flaws- for example, if activity at a previously popular citibike in FiDi decreased dramatically, we might see a really strong color that indicates low activity, but the radius of the marker might be so small that the viewer's eye might be drawn to something else. Of course, if there is a big cluster, this is remedied, but I think in a lot of instances important information is not conveyed. I also did not look into color theory or which colors pop most, which is pretty important in this type of analysis.

Anyway, I look forward to iterating on this and similar projects, and while the April dataset has not been published by Citibike yet, I think there is potential to [uncover totally new insights and patterns](https://www.outsideonline.com/2412755/more-people-cycling-coronavirus-pandemic).
