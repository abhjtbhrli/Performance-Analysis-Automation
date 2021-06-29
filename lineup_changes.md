# Automating Lineup Information 
### _Opposition Analysis 101_

This post is a continuation of the basis of [hacking out an UpSet plot](https://github.com/abhjtbhrli/Projects/blob/master/Upset%20Plot%20Hack.md) to analyse matchsheet data. In the previous post, I touched upon the basics of data loading, cleaning, wrangling and finally visualisation with `ggplot` using a dummy dataset. In here, real data is at work.

This article is the first of a series with supporting codes that will act as a step-by-step guide to automating the entire process of loading matchsheet data for any team and visualising it, part by part. Presenting matchsheet data is easy, but building a system to automate the process of loading, processing and presenting matchsheet data with one click makes the life of a performance analyst much easier.

The tasks in this part are made extremely easy by functions. We write two functions which will handle all aspects of connecting, loading, transforming and visualising.

Jason Zivkovic's `worldfootballR` package is of great help here in that it allows us to pull data from _transfermarkt.com_ with a few helper functions. Even though ISL data isn't as extensive and clean as big leagues' data on _transfermarkt.com_, matchsheet data is fairly consistent and since it's available to scrap and laod via `worldfootballR`, there's no point looking for other data sources.

To use the `worldfootballR` package, first install it ([instructions are here](https://github.com/JaseZiv/worldfootballR)) and then load the library with `library(worldfootballR)` command.

The first function we will write is the `teamsheet_data()` function. This function takes in an integer `season_end_year` (like 2021 for 2020-21 season) and a string `team_name` (like "Odisha" for Odisha FC) as arguments, and returns a dataframe containing the starting lineup details of every match of the given team's given season.

```
teamsheet_data <- function(season_end_year,team_name){
  library(worldfootballR)
  url<-as.data.frame(get_match_urls(country = "IND",gender = "M",season_end_year = season_end_year,tier = "1st"))
  names(url)<-"urls"
  url<-url%>%filter(str_detect(urls,"Indian-Super-League"))
  url<-url%>%filter(str_detect(urls,team_name))
  lineup<-get_match_lineups(url$urls)
  #lineup dataframe is ready
  
  lineup.team<-lineup%>%
    filter(str_detect(Team,team_name),Starting=="Pitch")%>%
    arrange(Matchday)
    
  #opponent & away formation dataframe
  
  lineup.meta<-lineup%>%
    group_by(Matchday)%>%
    dplyr::summarise(home=unique(Team)[1],
                     away=unique(Team)[2],
                     home_tactic=unique(Formation)[1],
                     away_tactic=ifelse(length(unique(Formation))==2L,
                                        unique(Formation)[2],unique(Formation)[1]))
  
  
  lineup.meta<-lineup.meta%>%
    mutate(vs.team=ifelse(home==team_name,away,home),
           vs.tactics=ifelse(home==team_name,away_tactic,home_tactic),
           match.num=c(1:length(unique(lineup.meta$Matchday))))
  
  lineup.allinfo<-merge(lineup.team,lineup.meta[,c("Matchday","vs.team","vs.tactics","match.num")],by="Matchday")
  return(lineup.allinfo)
}
```
The second funnction `start.changes()` uses the first function `teamsheet_data()` in a nested capacity. It generates a dataframe in the format of the dummy dataframe in [this post](https://github.com/abhjtbhrli/Projects/blob/master/Upset%20Plot%20Hack.md) so that we can plot the **UpSet plot** depicting the number of starting XI changes per game. The `start.changes()` function returns the resultant plot like this:


![Image](https://user-images.githubusercontent.com/37649445/123481688-b7274100-d621-11eb-99c8-ad3e179508fe.png)

The code chunk:

```
start.changes <- function(season_end_year,team_name){
  #use teamsheet_data() function to gather lineup info of particular team
  team_name_lineup<-teamsheet_data(season_end_year,team_name)
  #initialize an empty list to store lineups
  empty_list<-vector(mode = "list",length = length(unique(team_name_lineup$match.num)))
  #populate the empty list
  for (i in 1:length(unique(team_name_lineup$match.num))) {
    empty_list[i]=list(unique(team_name_lineup$Player_Name[which(team_name_lineup$match.num==i)]))
  }
  lineup_list<-as_tibble(empty_list,.name_repair = "unique")
  
  #make changes in the column names of the starting XI tibble (as column names are "...1", "...2",...
  #& we need better names)
  names(lineup_list)<-as.character(unique(team_name_lineup$match.num))
  
  #create dataframe for visualising changes data 
  #we create a "changes.df" dataframe with 3 columns - match, vs, changes
  
  #populate the match column
  changes.df<-data.frame(match=as.numeric(names(lineup_list)))
  #initialize empty vs & changes columns
  changes.df$vs<-character(nrow(changes.df))
  changes.df$changes<-integer(nrow(changes.df))
  
  #iterate through the vs column to populate it
  # for (i in 1:nrow(changes.df)) {
  #   changes.df$vs[i]=unique(lineup_list$vs.team[which(lineup_list$match.num==i)])
  # }
  
  #iterate through the changes column to populate it
  for (j in 2:(nrow(changes.df))) {
    changes.df$changes[j]=11-(length(intersect(as_vector(lineup_list[,j-1]),as_vector(lineup_list[,j]))))
  }
  
  #we can now plot the changes upset plot using ggplot
  
  changes.plot<-ggplot()+
    scale_x_continuous(breaks = seq(1,max(changes.df$match),1),position = "bottom")+
    scale_y_continuous(breaks = seq(1,max(changes.df$changes),1))+
    geom_segment(data = changes.df,
                 aes(x = match,y = 0,xend = match,yend = changes,colour = -changes),
                 size = 1,
                 alpha = 0.7,
                 show.legend = F)+
    geom_point(data = changes.df,
               aes(x = match,y = changes),
               size=2.5)+
    geom_point(data = changes.df,
               aes(x = match,y = 0),
               size = 2.5)+
    labs(title = "Starting XI Changes",
         subtitle = paste0(team_name_lineup$vs.team[str_detect(team_name_lineup$vs.team,team_name)][1]," \nISL ",season_end_year-1,"-",season_end_year))+
    xlab("Match no.")+
    ylab("No. of changes")+
    theme(
      text = element_text(family = "Courier"),
      plot.background = element_rect(fill="#F6FCF8"),
      panel.background = element_rect(fill = "#F6FCF8"),
      panel.grid = element_blank(),
      title = element_text(hjust = 0.5),
      plot.title = element_text(hjust = 0,face = "bold"),
      plot.subtitle = element_text(hjust = 0,size = 10)
    )
  return(changes.plot)
}
```

Now, we can generate **UpSet plot** of any ISL team from any season by keying in the `season_end_year` and the `team_name`.

Like this... `start.changes(season_end_year = 2021,team_name = "Mohun")`

... which gives us this:


![MB](https://user-images.githubusercontent.com/37649445/123698686-108aac80-d87c-11eb-9781-ead8cd33cd9c.png)


Fin.
