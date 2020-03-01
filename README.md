Computational Musicology Portfolio 

---
title: "Dashboard Pauline"
author: "Pauline Sayn-Wittgenstein" 
date: "February-March 2020"

output: 
  flexdashboard::flex_dashboard:
    orientation: columns
    vertical_layout: fill
    theme: flatly 
---

```{r setup, include=FALSE}
library(flexdashboard)
library(tidyverse)
library(plotly)
library(spotifyr)
library(compmus)
source('spotify.R')
```

Page 1
===================================== 

Column {data-width=500}
-----------------------------------------------------------------------

### Introduction 

I decided to focus on the difference between Reggeaton and German Hip Hop for my research. Therefore my final research question is:

How does the latin Hip Hop music style "Reggeaton" differ from German Hip Hop? 
Why is Reggeaton growing in popularity all around the world and German Hip Hop is not? Can we find specific features in Reggeaton music that might possibly cause its growing popularity? 

My first motivation for this research is my personal interest in both music styles. Additionally, I find it very impressive that Reggeaton's popularity is growing so rapidly. I personally know a lot of people who do not speak any Spanish but love to listen to Reggeaton music. Therefore, I find it very intersting to find out about that reason that makes people like Reggeaton so much.

Because after doing several analyses with different playlists created by Spotify and getting good suggestions from peers, I realized, it might be better to use playlists that include new but also old songs and not only the current Charts. I also decided not to use the playlists I created but rather use the most accurate ones I can find created by Spotify as this excludes my musical taste. Therefore, I do not project the tables and data from the analyses in this assignment but only the ones I created afterwards with the new playlists. 

The playlist I am using for the music style of Reggeaton is called "Reggeaton Classics" and includes 99 songs.
For the German Hip Hop I found a playlist by Spotify called "Deutschrap: Die Klassiker" but it only includes 50 songs. As I did not know if that is a problem I decided to create a new playlist including this and a second playlist by Spotify called "Modus Mio", and according to Spotify, it is the most important Hip-Hop playlists in Germany. I called this new playlist "German HipHop Classics". This playlist therefore ended with 100 songs. 

When looking at the means and standard deviations of both these playlists I could find small differences between the danceability, loudness and speechiness. 

The small differences that I could find from my first anaylses also reflect my expectations resulting from my own muscial experiences with both musical styles. I myself experience Reggeaton as very moving, danceable and often positive while German Hip Hop is often more negative and aggresive which might result in its loudness. Also, it seems to me that in the German songs I perceive much more text that is sung in a very fast way. Additionally, I perceive a very characteristic beat in most Reggeaton songs.
But of course there are also variations within each musical style and thats what I also saw when doing my first anaylses.


Row {.tabset .tabset-fade}
-----------------------------------------------------------------------

### Visualisation 1

```{r}

Reggeaton_username <- 'spotify'
German_username <- 'paulinchen1999'
Reggeaton_uris <- '37i9dQZF1DX8SfyqmSFDwe'
German_uris <- '3Q6aTzn71rrODobApwJlWR'

Reggeaton <- get_playlist_audio_features(Reggeaton_username, Reggeaton_uris)
German <- get_playlist_audio_features(German_username, German_uris)
both <- Reggeaton %>% mutate(playlist = "Reggeaton Classics") %>%bind_rows(German %>% mutate(playlist = "German HipHop Classics"))

genres <- both %>% mutate(mode=factor(mode,c(1,0),c("Major","Minor")))%>% ggplot(aes(x=valence, y=energy, size=loudness, colour=mode, label=track.name))+geom_point()+geom_rug(size=0.1)+facet_wrap(~playlist)+scale_x_continuous(limits=c(0,1),breaks=c(0, 0.50, 1),  minor_breaks = NULL) +scale_y_continuous(limits = c(0, 1), breaks = c(0, 0.50, 1),minor_breaks = NULL) +scale_colour_brewer(type="qual",palette="Paired")+scale_size_continuous(trans="exp",guide="none")+theme_light()+labs(x="Valence", y="Energy", colour="Mode")

ggplotly(genres)


``` 

### Interpretation 

Here is the first visualisation I created. It compares the valence on the x-axis to the energy of the song on the y-axis.From the first impression you do not really see a big difference between the German Hip-Hop and the Reggeaton songs. But looking closer, you can see that there is more variance between the German songs than within the Reggeaton songs. As I described in my own expectations, Reggeaton songs sound very positive and as the visualisation shows, German songs vary regarding valence. What we can also see is that both genres are quite energetic although I think this energy might be different for both of them. The energy in German songs seems more aggresive to me than the energy in the Reggeaton songs. This might also be visualised in the graph, as the German songs are also destributed more central than the Reggeaton songs. 


```{r}


```

Page 2
=====================================     

Column
-----------------------------------------------------------------------

### Chromagram - German song "Mios mit Bars" by Luciano

```{r}
Mios <- get_tidy_audio_analysis('7Ek9e3eIuktIFjpDRQfmHE') %>% select(segments) %>% unnest(segments) %>%  select(start, duration, pitches)

Mios %>% 
     mutate(pitches = map(pitches, compmus_normalise, 'euclidean')) %>% 
     compmus_gather_chroma %>% 
     ggplot(
         aes(
             x = start + duration / 2, 
             width = duration, 
             y = pitch_class, 
             fill = value)) + 
     geom_tile() +
     labs(x = 'Time (s)', y = NULL, fill = 'Magnitude') +
     theme_minimal()

```

### Chromagram - Reggeaton song "En la Cama" by Nicky Jam feat. Daddy Yankee

```{r}
Enlacama<- get_tidy_audio_analysis('2Eg6dOam7cAe5turf2bnCg') %>% 
     select(segments) %>% unnest(segments) %>% 
     select(start, duration, pitches)

Enlacama %>%  mutate(pitches = map(pitches, compmus_normalise, 'euclidean')) %>% 
     compmus_gather_chroma %>% 
     ggplot(
         aes(
             x = start + duration / 2, 
             width = duration, 
             y = pitch_class, 
             fill = value)) + 
     geom_tile() +
     labs(x = 'Time (s)', y = NULL, fill = 'Magnitude') +
     theme_minimal()

```

Column
-----------------------------------------------------------------------
### Dynamic Time Warping with Euclidean and method Cosine

```{r}
compmus_long_distance(
      Enlacama %>% mutate(pitches = map(pitches, compmus_normalise, 'euclidean')),
      Mios %>% mutate(pitches = map(pitches, compmus_normalise, 'euclidean')),
      feature = pitches,
      method = 'cosine') %>% 
      ggplot(
          aes(
              x = xstart + xduration / 2, 
              width = xduration,
              y = ystart + yduration / 2,
              height = yduration,
              fill = d)) + 
      geom_tile() +
      scale_fill_continuous(type = 'viridis', guide = 'none') +
      labs(x = 'En La Cama', y = 'Mios mit Bars') +
     theme_minimal()

```

### Dynamic Time Warping with Chebyshev and method euclidean

```{r}
compmus_long_distance(
    Enlacama %>% mutate(pitches = map(pitches, compmus_normalise, 'chebyshev')),Mios %>% mutate(pitches = map(pitches, compmus_normalise, 'chebyshev')),
    feature = pitches,
     method = 'euclidean') %>% 
     ggplot(
         aes(
             x = xstart + xduration / 2, 
             width = xduration,
             y = ystart + yduration / 2,
             height = yduration,
             fill = d)) + 
     geom_tile() +
     scale_fill_continuous(type = 'viridis', guide = 'none') +
     labs(x = 'En La Cama', y = 'Mios mit Bars') +
     theme_minimal()
```


Row 
-------------------------------------
    
### Conclusion

Until now, the visualisations I created showed very interesting results and I want to look further at audio features like danceability, loudness and tempo as a perceive those as very characteristic of both music styles. More importantly want to look more deeper and create more sophisticated visualisations.

What I can conclude from my visualisations until now, is that there might be significant differences between the two music styles, for instance in their level of energy and valence which might be a reason for why Reggeaton is also listened to be people who do not even speak Spanish. It might be it's positive, danceable and energetic sound that makes people like Reggeaton.
