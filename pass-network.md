# Automatically generate pass networks from InStat data

## Data mining in performance analysis

First, we have to write a function to autoclean the InStat dataframe of a particular match

```
passnetFuncNEW <- function(pass_net1,time1,time2){
  teams<-unique(pass_net1$team)
  
  pass_net2<-pass_net1%>%
    filter(team==teams[1],
           start>=time1*60 & start<=time2*60)%>%
    group_by(name,rec)%>%
    summarise(nPass=n())%>%
    arrange(desc(nPass))
  
  
  pass_net3<-pass_net1%>%
    filter(team==teams[1],
           start>=time1*60 & start<=time2*60)%>%
    group_by(name)%>%
    summarise(ave.x=mean(pos_x),ave.y=mean(pos_y))
  
  pass_net3$rec<-pass_net3$name
  
  lol<-merge(pass_net2,pass_net3[,c("ave.x","ave.y","name")],by = 'name')
  names(lol)<-c("name","rec","nPass","x","y")
  lol<-merge(lol,pass_net3[,c("ave.x","ave.y","rec")],by='rec')
  lol<-lol%>%select(name,rec,nPass,x,y,ave.x,ave.y)%>%arrange(desc(nPass))
  names(lol)<-c("name","rec","nPass","x","y","xend","yend")
  
  plot.passnet1<-ggplot()+
    annotate_pitch(dimensions = pitch_custom,fill = "#2A374A",colour = "#A6AAB0")+
    theme_pitch(aspect_ratio = 2/3)+
    geom_segment(data = lol%>%
                   filter(),
                 aes(x=x,y=y,xend=xend,yend=yend,alpha=nPass),
                 show.legend = F,
                 colour="#FB4447")+
    geom_point(data = pass_net3,
               aes(x=ave.x,y=ave.y))+
    geom_label(data = pass_net3%>%
                 separate(name,c("num","player"),". ",extra = "merge"),
               aes(x=ave.x,y=ave.y,label=num,family="Helvetica"),
               alpha=0.9,
               size=3)+
    labs(title = "PASS NETWORKS")+
    #labs(caption = toupper(teams[1]))+
    theme(text = element_text(family = "Helvetica"),
          plot.title = element_text(hjust = 0,
                                    size = 15,
                                    family = "Helvetica",
                                    colour = "#A6AAB0"),
          plot.caption = element_text(size = 10))+
    annotate("text",
             x=1,
             y=65,
             label=toupper(teams[1]),
             family="Helvetica",
             colour="#ffffff",
             size=3.5,
             hjust=0,
             vjust=0)+
    geom_segment(aes(x=0,y=-2.5,xend=30,yend=-2.5),size=1,lineend = "butt",
                 linejoin = "mitre",
                 arrow = arrow(type = "closed",length = unit(0.07,"inches")),
                 colour="#2A374A")+
    annotate("text",
             x=0,
             y=-5,
             label="attack direction",
             family="Helvetica",
             size=3.5,
             hjust=0,
             vjust=0)
     
     return(plot.passnet1)
     
 ```
 To extract just the dataframe for pass networks,
             
      
