---
layout: post
title: Exploratory analysis of the Global Terrorism Dataset with R
subtitle: Using tidyverse, ggplot2, and plotly
image: /img/GTD/TerrorThumbnail.jpg
gh-repo: eskilan/globalTerror
gh-badge: [star, fork, follow]
---

The following is an exploratory data analysis of the Global Terrorism Database (<http://www.start.umd.edu/gtd/about/>) curated by START, the national consortium for the study of terrorism and responses to terrorism and the University of Maryland, College Park. This data contains over 170,000 terrorist attacks, and is considered "the most comprehensive unclassified database on terrorist events in the world." 

![alt text](/img/GTD/Iraq-terrorist_attack_on_buses.jpg  "By Unknown, US Army image - https://web.archive.org/web/http://www4.army.mil/armyimages/armyimage.php?photo=7243, Public Domain, https://commons.wikimedia.org/w/index.php?curid=637415" ){:height="40%" width="40%"}

We will be exploring this dataset using the R language, using the ggplot2 package heavily to create statistical visualizations, and the ggplotly package to create interactive graphics.

``` r
library(tidyverse)
library(plotly)
library(maps)
```

``` r
gtd <- read_csv('globalterrorismdb_0617dist.csv') %>% as_tibble()
```

``` r
ggplot(data=gtd) + 
  geom_bar(mapping= aes(x=iyear)) + labs(x='Year',y='Number of terror incidents')
```

![](/img/GTD/unnamed-chunk-3-1.png)

We see that the number of attacks increased from 1970 to 1992. The year 1993 is missing, which is something the START acknowledges in its FAQ (<http://www.start.umd.edu/gtd/faq/>). We also notice an increase in incidents starting around 2003. Now let's take a look at the proportion of terror incidents on a per region basis:

``` r
perYear <- gtd %>% group_by(iyear,region_txt) %>% summarise(nIncidents=n())
ggplot(data=perYear) + 
  geom_area(mapping = aes(x=iyear,y=nIncidents,group=region_txt,fill=region_txt),position='fill') + 
  labs(x='Year',y='Number of terror incidents',fill='Region')
```

![](/img/GTD/unnamed-chunk-4-1.png)

We see some interesing trends. The number of terrorist incidents was very large in Western Europe from 1972 to 1980. We see how South America and Western Europe's proportion of terror incidents has decreased. We also see that the Middle East and North Africa together with South Asia carry the larger part of modern terror incidents.

### Western Europe

Next, we examine the periods of terrorism in western europe from 1970 to 1980 that show a strong period of turbulence:

``` r
we <- gtd %>% filter(region_txt == 'Western Europe',between(iyear,1970,1980)) # %>% group_by(country)
```

Now, we analyze the relationship between country, attack type, and number of victims:

``` r
# number of attacks per type and country
ggplot(data=we) + geom_count(mapping = aes(x=country_txt,y=attacktype1_txt)) + 
  theme(axis.text.x = element_text(angle = 60, hjust = 1)) +
  labs(x='Country',y='Attack type')
```

![](/img/GTD/unnamed-chunk-6-1.png)

We find that the UK, Italy, and Spain had many incidents, and France suffered bombings. Let's take a deeper look the the data. We will calculate the number of killed, and mean number of killed, per country and per attack type

``` r
weByMean <- we %>% group_by(country_txt,attacktype1_txt) %>% summarise(meanKills = mean(nkill,na.rm=TRUE))
weBySum <- we %>% group_by(country_txt,attacktype1_txt) %>% summarise(nKills = sum(nkill,na.rm=TRUE))
```

In terms of efficacy of attack type:

``` r
ggplot(data=weByMean,mapping = aes(x = country_txt, y = attacktype1_txt)) +
  geom_tile(mapping = aes(fill = meanKills)) +
  theme(axis.text.x = element_text(angle = 60, hjust = 1)) +
  labs(x='Country',y='Attack type', fill='Mean killed')
```

![](/img/GTD/unnamed-chunk-8-1.png)

In terms of total killed:

``` r
ggplot(data=weBySum,mapping = aes(x = country_txt, y = attacktype1_txt)) +
  geom_tile(mapping = aes(fill = nKills)) +
  theme(axis.text.x = element_text(angle = 60, hjust = 1)) + 
  labs(x='Country',y='Attack type', fill='Number of killed')
```

![](/img/GTD/unnamed-chunk-9-1.png)

What we learn from this is that despite having few killed per incident, the UK suffered the most losses. On the other hand, Hijackings were few but alse were the deadliest.

The most deadly type of terror incidents occurred in the UK as assasinations. Let's take a deeper look at UK assasinations and ask who commited the crimes and against whom?

#### UK assasinations

``` r
ukAssas <- filter(we,country_txt=='United Kingdom',attacktype1_txt == 'Assassination')
ggplot(data = ukAssas) + geom_count(mapping = aes(x=targtype1_txt,y=gname)) + 
  theme(axis.text.x = element_text(angle = 60, hjust = 1)) +
  labs(x='Type of target',y='Name of perpetrator group')
```

![](/img/GTD/unnamed-chunk-10-1.png) We see that the IRA attacked mostly police, Military targets and private citizens and property. Most other groups attacked private citizens and property. We can infer that the IRA was fighting against authority at the time.

### South America

Now let's switch topic and take a look at South America from 1970 to 2005:

``` r
sa <- filter(gtd,region_txt == 'South America',between(iyear,1970,2005))
```

Plotting country, attacktype, and number of victims

``` r
ggplot(data=sa) + geom_count(mapping = aes(x=country_txt,y=attacktype1_txt)) + 
  theme(axis.text.x = element_text(angle = 60, hjust = 1)) +
  labs(x='Country',y='Attack type')
```

![](/img/GTD/unnamed-chunk-12-1.png) We see that Colombia, Peru, and Chile come up in terms of number of incidents. However, which organizations killed the most people during this period?

``` r
saDeadliest <- sa %>% group_by(gname,country_txt) %>% 
  summarise(killed=sum(nkill)) %>% arrange(desc(killed)) %>% filter(killed>5)

ggplot(data = saDeadliest) + geom_tile(mapping = aes(x=country_txt,y=gname,fill=killed)) +
  theme(axis.text.x = element_text(angle = 60, hjust = 1)) +
  labs(x='Country',y='Name of perpetrator group',fill='Number of killed')
```

![](/img/GTD/unnamed-chunk-13-1.png)

From this tile plot we see that most South American terrorist organizations operated in Colombia. The Death Squad, AUC, and Paramilitaries were the deadliest. The presence of Hezbolla in Argentina is also an interesting finding.

### Middle East and North Africa

Now let's switch regions again and take a look at the middle east and North Africa from 1987 to 2016

``` r
mena <- filter(gtd,region_txt == 'Middle East & North Africa',between(iyear,1987,2016))
```

We plot country, attacktype, and number of victims

``` r
ggplot(data=mena) + geom_count(mapping = aes(x=country_txt,y=attacktype1_txt)) + 
  theme(axis.text.x = element_text(angle = 60, hjust = 1)) +
  labs(x='Country',y='Attack type')
```

![](/img/GTD/unnamed-chunk-15-1.png)

We see that Iraq has had a disproportionate amount of bombings and explosions. Let's look at the cities with the most bombings in this region where more than 200 people died: \#\#\#\# Bombings

``` r
menaBombs <- mena %>% filter(attacktype1_txt == 'Bombing/Explosion') %>%
  group_by(country_txt,city) %>% summarise(killed=sum(nkill)) %>% arrange(desc(killed)) %>% filter(killed>200)

ggplot(data=menaBombs) + 
  geom_col(mapping = aes(x=city,y=killed,fill=country_txt)) + 
  coord_flip() +
  labs(y='Number killed in bombing',x='City',fill='Country')
```

![](/img/GTD/unnamed-chunk-16-1.png)

We can see that the deadliest cities are all in Iraq. Two Turkish cities, Ankara and Istanbul also suffered many victims. The dataset also contains bombings in "unknown" city or cities in Egypt, and two Lebanese cities, Beirut and Tripoli.

### Suicide attacks

Let's now take a look at the number of suicide attacks around the world throughout time. We will also make this an interactive plot using plotly.

``` r
suicide <- gtd %>% filter(suicide==1) %>% group_by(iyear,country_txt) %>% summarise(n=n())
```

We look at suicide attacks from 1980 to 2016:

``` r
# converting years to categorical variables to create tile plot
yearBreaks <- 1980:2016
labels <- 1981:2016
suicideCat <- suicide %>% mutate(yearCategory=cut(iyear, breaks=yearBreaks,labels=labels))
# tile plot
ggplot(data=suicideCat,mapping = aes(x = yearCategory, y = country_txt)) +
  geom_tile(mapping = aes(fill = n)) +
  theme(axis.text.x = element_text(angle = 60, hjust = 1)) +
  labs(x='Year',y='Country',fill='Number of suicide attacks')
```

![](/img/GTD/unnamed-chunk-18-1.png) 
The vast amount of suicide attacks took place in Iraq between 2012 and 2016, and soon after after the US invasion. This was the period known as "the insurgency." We also see Afghanistan to a lesser degree, and also Nigeria. Let's take a closer look at the data: 

#### Suicide attacks in the Middle East and North Africa
Looking at the Middle East and North Africa and removing Iraq from plot since it's off the chart

``` r
suicide1 <- gtd %>% 
    filter(suicide==1,region_txt== 'Middle East & North Africa' & country_txt != 'Iraq') %>%
    group_by(iyear,country_txt) %>% summarise(n=n())
# creating a ggplot2 plot but not showing it
ggSuicide1 <- ggplot(data=suicide1) + 
    geom_line(mapping = aes(x=iyear,y=n,colour=country_txt,group=country_txt)) +
    guides(colour=FALSE) +
    labs(x='Year',y='Number of suicide attacks')
```

Showing an interactive plot:

<iframe height="600" id="igraph" scrolling="no" seamless="seamless" src="/interactive/suicideAttacksPlotly.html" width="850" frameBorder="0"></iframe> 


### A map of terror attacks involving suicide

``` r
world_map <- map_data("world")
# We create a base plot with gpplot2
p <- ggplot() + coord_fixed() +
  xlab("") + ylab("")
# Add map to base plot
base_world_messy <- p + geom_polygon(data=world_map, aes(x=long, y=lat, group=group), 
                                     colour="light green", fill="light green")
cleanup <- 
  theme(panel.grid.major = element_blank(), panel.grid.minor = element_blank(), 
        panel.background = element_rect(fill = 'white', colour = 'white'), 
        axis.line = element_line(colour = "white"), legend.position="none",
        axis.ticks=element_blank(), axis.text.x=element_blank(),
        axis.text.y=element_blank())

base_world <- base_world_messy + cleanup

suicideLoc <- gtd %>% filter(suicide==1) %>% group_by(latitude,longitude,city) %>% summarise(n=n(),killed=sum(nkill,na.rm=TRUE))

map_data <- 
  base_world +
  geom_point(data=suicideLoc, 
             aes(x=longitude, y=latitude), colour="Red", 
             fill="Pink",pch=21, size=1, alpha=I(0.7)) +
  labs(title='Suicide attacks from 1980 to 2016')

map_data
```

![](/img/GTD/unnamed-chunk-21-1.png)

That's it. I hope you found this analysis helpful. Let me know if you have any comments.

Thumbnail image: By 대한민국 국군 Republic of Korea Armed Forces [CC BY-SA 2.0 (https://creativecommons.org/licenses/by-sa/2.0)], via Wikimedia Commons