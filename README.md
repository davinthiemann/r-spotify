---
title: "Einflussfaktoren auf die Popularität von Spotify Songs"
author: "Marco Libera & Davin Thiemann"
date: "2024-10-03"
output:
  html_document:
    toc: yes 
    number_sections: yes
    theme: flatly
    code_folding: hide
  pdf_document:
    toc: yes
    toc_depth: '3'
---

```{r setup, include=FALSE}
knitr::opts_chunk$set(echo = TRUE)
```

# Einführung und Problemstellung
Die Musik-Streaming-Branche hat die Art und Weise, wie Songs an Popularität gewinnen und in den Charts platziert werden, revolutioniert. Plattformen wie Spotify spielen eine entscheidene Rolle für den Erfolg von Künstlern und ihren Songs. Durch die Analyse von Streaming-Daten können Muster und Trends identifiziert werden, die Aufschluss über die Faktoren geben, die zur Popularität eines Songs beitragen.

Ziel des Projekts ist es, den Spotify-Datensatz aus dem Jahr 2024 zu untersuchen und eine Reihe von Hypothesen zu überprüfen, um herauszufinden welche Faktoren die Beliebtheit eines Songs auf Spotify ausmachen und ein besseres Verständnis für die Einflussfaktoren zu erhalten. Dieser Datensatz enthält ausschließlich populäre und erfolgreiche Songs. Dies ist wichtig um die nachfolgende Analyse korrekt zu interpretieren. Die Analyse wird in R durchgeführt und in dem vorliegenden RMarkdown-Dokument zusammengefasst. Die Analyse wird durch klare Erklärungen unterstützt, um die angewendeten Methoden und deren Relevanz zu verdeutlichen. Außerdem werden im Anschluss zwei Modelle des maschinellen Lernens, der Entscheidungsbaum und das Neuronales Netz, grob vorgestellt.

Spezifisch werden folgende Hypothesen getestet:

1. Songs mit höherer Tanzbarkeit sind tendenziell beliebter
2. Kürzere Songs haben höhere Streaming-Zahlen als längere Songs
3. Songs die in vielen Playlists vorhanden sind haben mehr Streams
4. Songs mit expliziten Inhalten haben mehr Streams als Songs ohne expliziten Inhalt
5. Beliebte Songs auf Spotify sind auch auf anderen Plattformen erfolgreich
6. Songs mit höherem BPM sind tendenziell beliebter auf Streaming-Plattformen
7. Songs mit höherer Tanzbarkeit haben tendenziell auch höhere Energie-Werte

Für die Datenanalyse standen ursprünglich zwei Spotify-Datensätze von Kaggle zur Verfügung. Einer aus dem Jahr 2023 und einer aus dem Jahr 2024.

Kaggle Datensatz 2023: https://www.kaggle.com/datasets/nelgiriyewithana/top-spotify-songs-2023

Kaggle Datensatz 2024: https://www.kaggle.com/datasets/nelgiriyewithana/most-streamed-spotify-songs-2024

Da der Datensatz von 2024 jedoch umfangreicher ist, aber bestimmte Attribute enthält, die nur im Datensatz von 2023 vorhanden sind, wurden die fehlenden Attribute mithilfe der Spotify-API im Datensatz von 2024 ergänzt. Aufgrund dieser Erweiterung und der größeren Datenmenge des Datensatzes von 2024 wird der Datensatz von 2023 nicht weiter berücksichtigt.

Der Quellcode zu der Ergänzung musikalischer Attribute befindet sich im Abgabeordner. (spotify.py)  
<br>
<u>Erklärung zum python Code:</u>

Der Code importiert notwendige Bibliotheken und authentifiziert sich über die Spotify API mithilfe von SpotifyClientCredentials unter Verwendung der Client-ID und des Client-Secrets.
Der Datensatz von Kaggle (spotify_2024.csv) wird eingelesen (Spalte 'ISRC', da diese Variable eine einzigartige ID ist, die einem Lied zuzuordnen ist).
Die Funktion get_track_id_from_isrc verwendet die Spotify API, um die Spotify-Track-ID für jeden ISRC-Code zu finden.
Im Anschluss werden die Audio-Features mit der Funktion get_multiple_song_features abgerufen.
Die abgerufenen Track-IDs und Audio-Features werden mit dem ursprünglichen Datensatz (spotify_2024) gemerged, um ein erweitertes Datenset zu erstellen.
Das finale DataFrame mit den erweiterten Daten wird in einer neuen CSV-Datei (spotify_2024_final.csv) gespeichert und im Anschluss wieder zu spotify_2024 umbenannt.

<br>

# Datenvorbereitung
In diesem Abschnitt wird der Datensatz geladen, erkundet und bereinigt. Ziel ist es, eine saubere und nutzbare Datenbasis für die anschließende Analyse zu schaffen.

## Daten laden
Der mit dem obigen Python-Code überabeitete Datensatz wird im folgenden gemeinsam mit den für die Analyse benötigten Libraries geladen.  
Der Datensatz enthält die meistgestreamten Spotify-Songs des Jahres 2024.
```{r, include=FALSE}
# Laden aller benötigten Libraries

library(dplyr)
library(ggplot2)
library(tidyr)
library(scales)
library(corrplot)
library(cluster)
library(ggcorrplot)
library(rpart)
library(caret)
library(forecast)
library(pROC)
library(rpart.plot)
library(neuralnet)
```

```{r}
spotify_2024 <- read.csv("spotify_2024.csv")
```

## Daten erkunden
Im folgenden werden die Variablen des Datensatzes genauer betrachtet, um ein erstes Verständnis für die Art der Daten zu entwickeln.

In der Tabelle werden die Bedeutung der Variablen und die Variablentypen erklärt.

| Variable                     | Typ  | Bedeutung                                             | Range                     |
|:-----------------------------|:-----|:------------------------------------------------------|:--------------------------|
| **track_name**               | chr  | Name des Liedes                                       | -                         |
| **album_name**               | chr  | Name des Albums, zu dem das Lied gehört              | -                         |
| **artist**                   | chr  | Name des/der Künstler(s) des Liedes                  | -                         |
| **release_date**             | chr  | Veröffentlichungsdatum des Liedes im Format MM/DD/YYYY | -                         |
| **isrc**                     | chr  | International Standard Recording Code des Liedes       | -                         |
| **all_time_rank**            | chr  | Alltime-Rang des Liedes                               | -                         |
| **track_score**              | num  | Punktzahl des Liedes                                   | 0 bis 100                 |
| **spotify_streams**          | chr  | Gesamte Anzahl der Streams auf Spotify                 | 0 bis ∞                   |
| **spotify_playlist_count**    | chr  | Anzahl der Spotify-Playlists, in denen das Lied enthalten ist | 0 bis ∞              |
| **spotify_playlist_reach**    | chr  | Reichweite der Spotify-Playlists, in denen das Lied enthalten ist | 0 bis ∞            |
| **spotify_popularity**       | num  | Beliebtheit des Liedes auf Spotify                    | 0 bis 100                 |
| **youtube_views**            | chr  | Anzahl der Aufrufe des Liedes auf YouTube             | 0 bis ∞                   |
| **youtube_likes**            | chr  | Anzahl der Likes des Liedes auf YouTube               | 0 bis ∞                   |
| **tiktok_posts**             | chr  | Anzahl der TikTok-Posts, die das Lied verwenden        | 0 bis ∞                   |
| **tiktok_likes**             | chr  | Anzahl der Likes des Liedes auf TikTok                 | 0 bis ∞                   |
| **tiktok_views**             | chr  | Anzahl der Aufrufe des Liedes auf TikTok               | 0 bis ∞                   |
| **youtube_playlist_reach**    | chr  | Reichweite der YouTube-Playlists, in denen das Lied enthalten ist | 0 bis ∞            |
| **apple_music_playlist_count**| num  | Anzahl der Apple Music-Playlists, in denen das Lied enthalten ist | 0 bis ∞              |
| **airplay_spins**            | chr  | Anzahl der AirPlay-Aufrufe des Liedes                  | 0 bis ∞                   |
| **siriusxm_spins**           | chr  | Anzahl der Aufrufe des Liedes auf SiriusXM            | 0 bis ∞                   |
| **deezer_playlist_count**    | num  | Anzahl der Deezer-Playlists, in denen das Lied enthalten ist | 0 bis ∞              |
| **deezer_playlist_reach**    | chr  | Reichweite der Deezer-Playlists, in denen das Lied enthalten ist | 0 bis ∞            |
| **amazon_playlist_count**     | num  | Anzahl der Amazon-Playlists, in denen das Lied enthalten ist | 0 bis ∞              |
| **pandora_streams**          | chr  | Anzahl der Streams des Liedes auf Pandora              | 0 bis ∞                   |
| **pandora_track_stations**   | chr  | Anzahl der Pandora Stationen, die das Lied spielen      | 0 bis ∞                   |
| **soundcloud_streams**       | chr  | Anzahl der Streams des Liedes auf Soundcloud           | 0 bis ∞                   |
| **shazam_counts**            | chr  | Anzahl der Shazam Abfragen für das Lied                | 0 bis ∞                   |
| **tidal_popularity**         | logi | Beliebtheit des Liedes auf TIDAL (nicht vorhanden = NA) | 0 bis 100               |
| **explicit_track**           | int  | Gibt an, ob das Lied als explizit gekennzeichnet ist (1 = ja, 0 = nein) | 0 oder 1         |
| **track_id**                 | chr  | Eindeutige ID des Liedes                               | -                         |
| **bpm**                      | num  | Beats per Minute, eine Messung des Liedtempos         | 0 bis 300                 |
| **key**                      | num  | Tonart des Liedes                                      | 0 bis 11                  |
| **mode**                     | num  | Modalität des Liedes (Dur oder Moll)                   | 0 oder 1                  |
| **danceability**             | num  | Eignung des Liedes zum Tanzen. Skala zwischen 0 und 1. Je näher an 1 desto "tanzbarer" ist das Lied | 0 bis 1              |
| **valence**                  | num  | Beschreibt die von einem Titel vermittelte musikalische Positivität. Tracks mit hoher Valenz klingen positiver, während Tracks mit niedriger Valenz eher negativ klingen | 0 bis 1             |
| **energy**                   | num  | beschreibt die Intensität und Aktivität eines Liedes    | 0 bis 1                   |
| **acousticness**             | num  | Anteil akustischer Elemente im Lied. Je höher desto wahrscheinlicher ist es, dass der Track akustisch ist | 0 bis 1        |
| **instrumentalness**         | num  | Sagt voraus, ob ein Track keinen Gesang enthält. Je näher der Wert für die Instrumentalität bei 1,0 liegt, desto größer ist die Wahrscheinlichkeit, dass der Titel keinen Gesang enthält | 0 bis 1     |
| **liveness**                 | num  | Erkennt die Anwesenheit eines Publikums in der Aufnahme. Höhere Liveness-Werte bedeuten eine höhere Wahrscheinlichkeit, dass der Track live gespielt wurde | 0 bis 1    |
| **speechiness**              | num  | Anteil der gesprochenen Wörter im Lied                  | 0 bis 1                   |
| **duration_ms**              | num  | Dauer des Liedes in Millisekunden                      | 0 bis ∞                   |


Genauer kann die Bedeutung der verschiedenen Variablen bei der Spotify API Dokumentation nachgeschaut werden: https://developer.spotify.com/documentation/web-api/reference/get-audio-features

Für die kommende Analyse ist es wichtig zu erwähnen, dass die Variable Spotify-Popularität von 0 bis 100 angibt, wie beliebt ein Song auf der Plattform ist.

```{r}
str(spotify_2024)
head(spotify_2024)
```

## Nicht benötigte Variablen löschen
In dieser Phase werden Variablen aus dem Datensatz entfernt, die für die geplanten Analysen nicht erforderlich sind. Hierfür wird das dplyr-Paket verwendet.
```{r}
spotify_2024 <- spotify_2024 %>%
  select(-ISRC, -Pandora.Track.Stations, -SiriusXM.Spins, -AirPlay.Spins, -TikTok.Posts, -TIDAL.Popularity, -Amazon.Playlist.Count, -id_x, -id_y)
```

## Anpassung der Datentypen
Nach dem Laden der Daten und dem Entfernen unnötiger Variablen folgt die Anpassung der Datentypen. Viele Variablen, die numerisch sein sollten, sind als Zeichenketten gespeichert, was auf Formatierungsprobleme zurückzuführen ist. Für eine genaue Analyse müssen diese in numerische Typen umgewandelt werden. Die Variable Release.Date wurde im Format Tag/Monat/Jahr formatiert, und die Variablen danceability, valence, energy, acousticness, instrumentalness, liveness und speechiness als Prozentwerte definiert.

```{r}
clean_numeric <- function(x) {
  as.numeric(gsub(",", "", x))
}
spotify_2024$Spotify.Streams <- clean_numeric(spotify_2024$Spotify.Streams)
spotify_2024$Spotify.Playlist.Count <- clean_numeric(spotify_2024$Spotify.Playlist.Count)
spotify_2024$Spotify.Playlist.Reach <- clean_numeric(spotify_2024$Spotify.Playlist.Reach)
spotify_2024$YouTube.Views <- clean_numeric(spotify_2024$YouTube.Views)
spotify_2024$YouTube.Likes <- clean_numeric(spotify_2024$YouTube.Likes)
spotify_2024$TikTok.Likes <- clean_numeric(spotify_2024$TikTok.Likes)
spotify_2024$TikTok.Views <- clean_numeric(spotify_2024$TikTok.Views)
spotify_2024$YouTube.Playlist.Reach <- clean_numeric(spotify_2024$YouTube.Playlist.Reach)
spotify_2024$Deezer.Playlist.Reach <- clean_numeric(spotify_2024$Deezer.Playlist.Reach)
spotify_2024$Pandora.Streams <- clean_numeric(spotify_2024$Pandora.Streams)
spotify_2024$Soundcloud.Streams <- clean_numeric(spotify_2024$Soundcloud.Streams)
spotify_2024$Shazam.Counts <- clean_numeric(spotify_2024$Shazam.Counts)
spotify_2024$Release.Date <- as.Date(spotify_2024$Release.Date, format = "%m/%d/%Y")
spotify_2024$Release.Date <- format(spotify_2024$Release.Date, "%d/%m/%Y")
spotify_2024$duration_seconds <- spotify_2024$duration_ms / 1000

percentage_columns <- c("danceability", "valence", "energy", "acousticness", "instrumentalness", "liveness", "speechiness")
spotify_2024[percentage_columns] <- lapply(spotify_2024[percentage_columns], function(x) x * 100)

str(spotify_2024[c("Spotify.Streams", "Spotify.Playlist.Count", "YouTube.Views", "TikTok.Views", "Release.Date", percentage_columns, "duration_seconds")])

```

## Daten bereinigen
Die Überprüfung auf fehlende Werte ist entscheidend, um die Qualität der Analysen sicherzustellen. Fehlende Daten wurden mit der Funktion na.omit() behandelt, welche Zeilen mit fehlenden Werten aus dem Datensatz entfernt. Auch Ausreißer werden in diesem Schritt bereinigt. Dies stellt sicher, dass die folgenden Analysen auf einem vollständigen und repräsentativen Datensatz basieren.
```{r}
na_counts <- sapply(spotify_2024, function(x) sum(is.na(x)))
print(na_counts)

spotify_2024 <- na.omit(spotify_2024)

spotify_2024 <- spotify_2024[!duplicated(spotify_2024), ]

boxplot_stats <- boxplot.stats(spotify_2024$Spotify.Streams)$out
spotify_2024 <- spotify_2024[!spotify_2024$Spotify.Streams %in% boxplot_stats, ]
```

<br>

# Explorative Datenanylse
Die explorative Datenanalyse dient dazu, die Daten vor der eigentlichen Analyse tiefergehend zu verstehen und wesentliche Merkmale und Muster zu erkennen, die Einfluss auf die Ergebnisse haben könnten. In diesem Abschnitt untersuchen wir Verteilungen von Schlüsselvariablen, identifizieren Ausreißer, prüfen Zusammenhänge zwischen den Merkmalen und bereiten die Daten visuell auf, um ein intuitives Verständnis für den Datensatz zu entwickeln.

Deskriptive Statistiken bieten einen Überblick über die zentralen Tendenzen und die Verteilung der numerischen Variablen im Datensatz:
```{r}
numerical_stats <- spotify_2024 %>%
  select(Spotify.Streams, YouTube.Views, TikTok.Views, bpm, danceability, energy, acousticness, valence, liveness, speechiness) %>%
  summarise_all(list(
    mean = ~mean(., na.rm = TRUE),
    median = ~median(., na.rm = TRUE),
    sd = ~sd(., na.rm = TRUE),
    min = ~min(., na.rm = TRUE),
    max = ~max(., na.rm = TRUE)
  ))

print(numerical_stats)
```

<br>
<br>  
**<u>Top 20 Songs nach Spotify-Streams:</u>**  
Übersicht über die 20 meist gestreamten Songs auf Spotify
```{r}

top20_songs <- spotify_2024 %>%
arrange(desc(Spotify.Streams)) %>%
slice_head(n = 20)

ggplot(top20_songs, aes(y = reorder(Track, -Spotify.Streams), x = Spotify.Streams)) +
geom_bar(stat = "identity", fill = "#1DB954") +
labs(title = "", x = "Streams", y = "Song") +
scale_x_continuous(labels = scales::label_number()) +
theme(axis.text.y = element_text(size = 7))
```
  
<br>
<br>  
**<u>Top 20 Künstler nach Spotify-Streams:</u>**  
Übersicht der 20 meistgestreamten Künstler auf Spotify
```{r}
top20_artists <- spotify_2024 %>%
  group_by(Artist) %>%
  summarise(Total.Streams = sum(Spotify.Streams)) %>%
  arrange(desc(Total.Streams)) %>%
  slice_head(n = 20)

ggplot(top20_artists, aes(x = reorder(Artist, -Total.Streams), y = Total.Streams)) +
  geom_bar(stat = "identity", fill = "#1DB954") +
  labs(title = "", x = "Künstler", y = "Streams") +
  theme(axis.text.x = element_text(angle = 90, hjust = 1)) + 
  scale_y_continuous(labels = scales::label_number()) 

```

<br>  
**<u>Beziehung zwischen der Popularität und musikalischen Eigenschaften</u>**

Im folgenden werden die musikalischen Eigenschaften mit der Variable 'Spotify.Popularity' verglichen. Dadurch können möglicherweise Muster oder Zusammenhänge erkannt werden.

<br>
<u>Beziehung zwischen 'danceability' und 'Spotify-Popularity'</u>
```{r}
ggplot(spotify_2024, aes(x = danceability, y = Spotify.Popularity)) +
  geom_point(color = "#1DB954", alpha = 0.5) +
  labs(title = "", x = "Danceability", y = "Popularity")
```

Die Punkte in der Abbildung sind gleichmäßig verteilt und deuten darauf hin, dass die 'danceability' keinen starken Einfluss auf die Popularität hat.

<br>
<u>Beziehung zwischen 'energy' und 'Spotify-Popularity'</u>
```{r}
ggplot(spotify_2024, aes(x = energy, y = Spotify.Popularity)) +
   geom_point(color = "#1DB954", alpha = 0.5) +
  labs(title = "", x = "Energy", y = "Popularity") +
  theme_minimal()
```

Auch in dieser Abbildung sind die Punkte gleichmäßig verteilt und deuten darauf hin, dass die 'energy' keinen starken Einfluss auf die Popularität hat

<br>
<u>Beziehung zwischen 'key' und 'Spotify-Popularity'</u>
```{r}
ggplot(spotify_2024, aes(x = key, y = Spotify.Popularity)) +
  geom_point(color = "#1DB954", alpha = 0.5) +
  labs(title = "", x = "Key", y = "Popularity") +
  theme_minimal()
```

Die Punkte verteilen sich ohne erkennbares Muster, was darauf hinweist, dass die Tonart (Key) keinen wesentlichen Einfluss auf die Popularität hat.

<br>
<u>Beziehung zwischen 'BPM' und 'Spotify-Popularity'</u>
```{r}
ggplot(spotify_2024, aes(x = bpm, y = Spotify.Popularity)) +
  geom_point(color = "#1DB954", alpha = 0.5) +
  labs(title = "", x = "BPM", y = "Popularity") +
  theme_minimal()
```

Die Verteilung der Punkte zeigt keine starke Korrelation zwischen der Beats per Minute (BPM) und der Popularität, was darauf schließen lässt, dass der Song-Tempo keinen großen Einfluss hat.

<br>
<u>Beziehung zwischen 'valence' und 'Spotify-Popularity'</u>
```{r}
ggplot(spotify_2024, aes(x = valence, y = Spotify.Popularity)) +
  geom_point(color = "#1DB954", alpha = 0.5) +
  labs(title = "", x = "Valence", y = "Popularity") +
  theme_minimal()
```

'Valence', das fröhliche oder traurige Stimmungen eines Songs misst, zeigt ebenfalls keine signifikante Beziehung zur Popularität, da die Punkte im Diagramm zufällig verteilt sind.

<br>
<u>Beziehung zwischen 'acousticness' und 'Spotify-Popularity'</u>
```{r}
ggplot(spotify_2024, aes(x = acousticness, y = Spotify.Popularity)) +
  geom_point(color = "#1DB954", alpha = 0.5) +
  labs(title = "", x = "Acousticness", y = "Popularity") +
  theme_minimal()
```

Es fällt auf, dass viele Punkte im unteren Bereich von 'acousticness' und im hohen Bereich der Popularity liegen, was darauf hindeuten könnte, dass Songs mit einem geringeren Anteil an 'acousticness' eine hohe Popularität aufweisen.

<br>
<u>Beziehung zwischen 'instrumentalness' und 'Spotify-Popularity'</u>
```{r}
ggplot(spotify_2024, aes(x = instrumentalness, y = Spotify.Popularity)) +
  geom_point(color = "#1DB954", alpha = 0.5) +
  labs(title = "", x = "Instrumentalness", y = "Popularity") +
  theme_minimal()
```

Es ist eine auffällige Ansammlung von Punkten bei 0 für die Instrumentalität und im Bereich von 40 bis 85 für die Beliebtheit zu beobachten. Dies deutet darauf hin, dass Songs mit niedriger Instrumentalität tendenziell beliebter sind. Dies impliziert, dass Lieder mit Gesang allgemein eine höhere Beliebtheit aufweisen.

<br>
<u>Beziehung zwischen 'liveness' und 'Spotify-Popularity'</u>
```{r}
ggplot(spotify_2024, aes(x = liveness, y = Spotify.Popularity)) +
  geom_point(color = "#1DB954", alpha = 0.5) +
  labs(title = "", x = "Liveness", y = "Popularity") +
  theme_minimal()
```

Es zeigt sich eine Häufung der Punkte bei etwa 15 für die 'liveness' und 75 für die Beliebtheit. Dies deutet darauf hin, dass Songs mit geringeren Live-Charakter bei den Nutzern beliebter sind.

<br>
<u>Beziehung zwischen 'speechiness' und 'Spotify-Popularity'</u>
```{r}
ggplot(spotify_2024, aes(x = speechiness, y = Spotify.Popularity)) +
  geom_point(color = "#1DB954", alpha = 0.5) +
  labs(title = "", x = "Speechiness", y = "Popularity") +
  theme_minimal()
```

Viele Punkte sind im Bereich von ca. 5 für 'speechiness' und 80 für Popularity, was darauf hindeutet, dass Songs die sowohl aus gesprochenen Wörtern und Musik bestehen tendenziell beliebter sind als Songs die nur aus Wörtern bestehen (z.B. Gedichte).

<br>
<u>Beziehung zwischen 'duration_seconds' und 'Spotify-Popularity'</u>
```{r}
ggplot(spotify_2024, aes(x = duration_seconds, y = Spotify.Popularity)) +
  geom_point(color = "#1DB954", alpha = 0.5) +
  labs(title = "", x = "Duration (Seconds)", y = "Popularity") +
  theme_minimal()
```

Eine Häufung der Punkte ist bei 150-250 Sekunden für die Dauer und 75 für Popularity zu beobachten, was darauf hindeutet, dass Songs mit mittlerer Länge häufiger eine höhere Popularität erreichen.

<br>
<br>  
**<u>Anzahl Streams der verschiedenen Platformen</u>**  
Übersicht der Gesamtanzahl an Streams gruppiert nach den im Datensatz vorhandenen Plattformen: TikTok, Spotify, YouTube, Pandora, Soundcloud und Deezer.
```{r}

platforms <- spotify_2024 %>%
summarise(
Spotify = sum(Spotify.Streams, na.rm = TRUE),
YouTube = sum(YouTube.Views, na.rm = TRUE),
TikTok = sum(TikTok.Views, na.rm = TRUE),
Deezer = sum(Deezer.Playlist.Reach, na.rm = TRUE),
Pandora = sum(Pandora.Streams, na.rm = TRUE),
Soundcloud = sum(Soundcloud.Streams, na.rm = TRUE)
) %>%
pivot_longer(everything(), names_to = "Platform", values_to = "Streams")

ggplot(platforms, aes(x = reorder(Platform, -Streams), y = Streams)) +
geom_bar(stat = "identity", fill = "#1DB954") +
labs(title = "", x = "Plattform", y = "Streams") +
scale_y_continuous(labels = label_comma()) +  
theme(axis.text.x = element_text(angle = 45, hjust = 1))
```
  
<br>
<br>  

**<u>Korrelation zwischen Audio Features</u>**  
Übersicht über die Korrelationen der im Datensatz vorhandenen musikalischen Merkmale.
```{r}

cor_matrix <- spotify_2024 %>%
  select(danceability, energy, valence, acousticness, instrumentalness, liveness, speechiness, bpm) %>%
  cor()

corrplot(cor_matrix, method = "circle", 
         col = colorRampPalette(c("red", "white", "#1DB954"))(200),
         tl.col = "black")
```
**<u>Danceability und Energy (0.09):</u>**

* Diese geringe positive Korrelation bedeutet, dass Songs mit höherer Tanzbarkeit tendenziell auch etwas mehr Energie haben.

**<u>Danceability und Valence (0.40):</u>**

* Diese moderate positive Korrelation zeigt, dass tanzbare Songs häufiger als "positiv" oder "fröhlich" wahrgenommen werden (gemessen durch Valence).

**<u>Danceability und Acousticness (-0.23):</u>**

* Die negative Korrelation zeigt, dass tanzbare Songs in der Regel weniger akustische Elemente enthält.

**<u>Energy und Valence (0.37):</u>**

* Es gibt eine moderate positive Korrelation zwischen Energie und Valence. Das deutet darauf hin, dass energetische Songs tendenziell auch positivere Stimmungen vermitteln.

**<u>Energy und Acousticness (-0.51):</u>**

* Diese starke negative Korrelation zeigt, dass Songs mit hoher Energie normalerweise weniger akustisch sind.

**<u>Speechiness und Danceability (0.26):</u>**

* Eine moderate positive Korrelation zeigt, dass Songs mit mehr gesprochenem Textanteil (Speechiness) tendenziell tanzbarer sind.

**<u>Instrumentalness und Energy (-0.01):</u>**

* Die fast nicht vorhandene Korrelation zeigt, dass es praktisch keinen Zusammenhang zwischen dem instrumentalen Anteil eines Songs und seiner Energie gibt.

**<u>Liveness und Energy (0.16):</u>**

* Diese schwache positive Korrelation bedeutet, dass Songs, die eine Live-Atmosphäre vermitteln (Liveness), tendenziell auch etwas energiegeladener sind.

**<u>Acousticness und Valence (-0.11):</u>**

* Diese schwache negative Korrelation deutet darauf hin, dass akustischere Songs tendenziell weniger "glücklich" klingen. Akustik könnte also mit melancholischeren oder nachdenklicheren Stimmungen assoziiert werden.

**<u>BPM und Energy (0.11):</u>**

* Die leichte positive Korrelation zwischen Beats per Minute (BPM) und Energie zeigt, dass schnellere Songs tendenziell etwas energiegeladener sind, auch wenn dieser Zusammenhang nur schwach ist.

Zusammenfassend zeigen diese Korrelationen, dass tanzbare und energetische Songs tendenziell positiver und weniger akustisch sind, während akustische Songs tendenziell eine ruhigere und weniger energetische Stimmung vermitteln.

<br>  
**<u>Clusteranalyse - Gruppierung der Songs basierend auf ihren Merkmalen</u>**  
Für diese Clusteranalyse werden die folgenden Variablen betrachtet:  

* danceability, energy, valence, acousticness, instrumentalness, liveness, speechiness, bpm, Spotify.Streams, Spotify.Playlist.Count, Spotify.Popularity, YouTube.Views, YouTube.Likes, TikTok.Likes, TikTok.Views, duration_seconds
```{r}
set.seed(187)

kmeans_result <- kmeans(spotify_2024 %>% 
                          select(danceability, energy, valence, acousticness, instrumentalness, liveness, 
                                 speechiness, bpm, Spotify.Streams, Spotify.Playlist.Count, Spotify.Popularity, 
                                 YouTube.Views, YouTube.Likes, TikTok.Likes, TikTok.Views, duration_seconds), 
                        centers = 3)

spotify_2024$Cluster <- as.factor(kmeans_result$cluster)

ggplot(spotify_2024, aes(x = danceability, y = energy, color = Cluster)) +
  geom_point() +
  labs(title = "Cluster von Songs basierend auf Danceability und Energy", x = "Danceability", y = "Energy")
```

-  Cluster 1: Tanzbare, energiegeladene Songs, beliebt auf Spotify und YouTube
-  Cluster 2: Songs mit moderater Tanzbarkeit, besonders erfolgreich auf TikTok
-  Cluster 3: Ruhigere, weniger tanzbare Songs, mäßig erfolgreich auf verschiedenen Plattformen

<br>
<br>

**<u>Clusteranalyse - Gruppierung der Songs basierend auf ihren Popularitätsmetriken</u>**

Für diese Clusteranalyse werden die folgenden Variablen betrachtet:

* Spotify.Streams, Spotify.Playlist.Count, Spotify.Popularity, YouTube.Views, YouTube.Likes, TikTok.Likes, TikTok.Views
```{r}
set.seed(187)

features <- spotify_2024 %>%
  select(Spotify.Streams, Spotify.Playlist.Count, Spotify.Popularity, 
         YouTube.Views, YouTube.Likes, TikTok.Likes, TikTok.Views)

kmeans_result <- kmeans(features, centers = 3)

spotify_2024$Cluster <- as.factor(kmeans_result$cluster)

ggplot(spotify_2024, aes(x = Spotify.Streams, y = YouTube.Views, color = Cluster)) +
  geom_point() +
  labs(title = "Cluster von Songs basierend auf Popularitätsmetriken", 
       x = "Spotify Streams", 
       y = "YouTube Views") +
  scale_x_continuous(labels = scales::label_number(big.mark = ",", decimal.mark = ".")) +
  scale_y_continuous(labels = scales::label_number(big.mark = ",", decimal.mark = "."))
```

-  Cluster 1: Songs, die auf Spotify sehr beliebt sind, aber auf YouTube weniger erfolgreich
-  Cluster 2: Songs, die auf TikTok besonders populär sind
-  Cluster 3: Songs, die auf mehreren Plattformen durchschnittlich populär sind

<br>
<br>
**<u>Histogramme</u>**  
Die folgenden Histogramme zeigen, wie häufig die Werte der Variablen im Datensatz vorkommen.
```{r}

ggplot(spotify_2024, aes(x = Spotify.Streams)) +
geom_histogram(binwidth = 50000000, fill = "#1DB954", color = "black") +
labs(title = "Verteilung der Spotify Streams", x = "Spotify Streams", y = "Häufigkeit") +
scale_x_continuous(labels = label_number(big.mark = ","))

ggplot(spotify_2024, aes(x = danceability)) +
geom_histogram(binwidth = 10, fill = "#1DB954", color = "black") +
labs(title = "Verteilung der Danceability", x = "Danceability", y = "Häufigkeit")

ggplot(spotify_2024, aes(x = energy)) +
geom_histogram(binwidth = 10, fill = "#1DB954", color = "black") +
labs(title = "Verteilung der Energy", x = "Energy", y = "Häufigkeit")

ggplot(spotify_2024, aes(x = Spotify.Popularity)) +
geom_histogram(binwidth = 5, fill = "#1DB954", color = "black") +
labs(title = "Verteilung der Spotify Popularity", x = "Spotify Popularity", y = "Häufigkeit")

ggplot(spotify_2024, aes(x = duration_seconds)) +
geom_histogram(binwidth = 30, fill = "#1DB954", color = "black") +
labs(title = "Verteilung der Songdauer in Sekunden", x = "Songdauer (Sekunden)", y = "Häufigkeit")

ggplot(spotify_2024, aes(x = Spotify.Playlist.Count)) +
geom_histogram(binwidth = 10000, fill = "#1DB954", color = "black") +
labs(title = "Verteilung der Spotify Playlist Count", x = "Spotify Playlist Count", y = "Häufigkeit") +
scale_x_continuous(labels = label_number(big.mark = ","))

ggplot(spotify_2024, aes(x = YouTube.Views)) +
geom_histogram(binwidth = 50000000, fill = "#1DB954", color = "black") +
labs(title = "Verteilung der YouTube Views", x = "YouTube Views", y = "Häufigkeit")  +
scale_x_continuous(labels = label_number(big.mark = ","))

ggplot(spotify_2024, aes(x = YouTube.Likes)) +
geom_histogram(binwidth = 1000000, fill = "#1DB954", color = "black") +
labs(title = "Verteilung der YouTube Likes", x = "YouTube Likes", y = "Häufigkeit") +
scale_x_continuous(labels = label_number(big.mark = ","))

ggplot(spotify_2024, aes(x = TikTok.Likes)) +
geom_histogram(binwidth = 50000000, fill = "#1DB954", color = "black") +
labs(title = "Verteilung der TikTok Likes", x = "TikTok Likes", y = "Häufigkeit") +
scale_x_continuous(labels = label_number(big.mark = ","))

ggplot(spotify_2024, aes(x = TikTok.Views)) +
geom_histogram(binwidth = 500000000, fill = "#1DB954", color = "black") +
labs(title = "Verteilung der TikTok Views", x = "TikTok Views", y = "Häufigkeit") +
scale_x_continuous(labels = label_number(big.mark = ","))
```

Die dargestellten Histogramme bieten einen umfassenden Überblick über die Verteilung der verschiedenen Variablen im Datensatz, verdeutlichen die Häufigkeit unterschiedlicher Werte und ermöglichen zudem die Erkennung von Ausreißern.
  
<br>
<br>
**<u>Boxplots</u>**  
Die Verteilung der Daten wird grafisch dargestellt. Es werden die Quartile, der Median und nochmals mögliche Ausreißer identifiziert.  
```{r}
ggplot(spotify_2024, aes(y = Spotify.Streams)) +
  geom_boxplot(fill = "#1DB954") +
  labs(title = "Boxplot der Spotify Streams", y = "Spotify Streams") +
  scale_y_continuous(labels = label_number(big.mark = ",", decimal.mark = "."))

ggplot(spotify_2024, aes(y = danceability)) +
  geom_boxplot(fill = "#1DB954") +
  labs(title = "Boxplot der Danceability", y = "Danceability") +
  scale_y_continuous(labels = label_number(big.mark = ",", decimal.mark = "."))

ggplot(spotify_2024, aes(y = energy)) +
  geom_boxplot(fill = "#1DB954") +
  labs(title = "Boxplot der Energy", y = "Energy") +
  scale_y_continuous(labels = label_number(big.mark = ",", decimal.mark = "."))

ggplot(spotify_2024, aes(y = Spotify.Popularity)) +
  geom_boxplot(fill = "#1DB954") +
  labs(title = "Boxplot der Spotify Popularity", y = "Spotify Popularity") +
  scale_y_continuous(labels = label_number(big.mark = ",", decimal.mark = "."))

ggplot(spotify_2024, aes(y = duration_seconds)) +
  geom_boxplot(fill = "#1DB954") +
  labs(title = "Boxplot der Songdauer in Sekunden", y = "Songdauer (Sekunden)") +
  scale_y_continuous(labels = label_number(big.mark = ",", decimal.mark = "."))

ggplot(spotify_2024, aes(y = Spotify.Playlist.Count)) +
  geom_boxplot(fill = "#1DB954") +
  labs(title = "Boxplot der Spotify Playlist Count", y = "Spotify Playlist Count") +
  scale_y_continuous(labels = label_number(big.mark = ",", decimal.mark = "."))

ggplot(spotify_2024, aes(y = YouTube.Views)) +
  geom_boxplot(fill = "#1DB954") +
  labs(title = "Boxplot der YouTube Views", y = "YouTube Views") +
  scale_y_continuous(labels = label_number(big.mark = ",", decimal.mark = "."))

ggplot(spotify_2024, aes(y = YouTube.Likes)) +
  geom_boxplot(fill = "#1DB954") +
  labs(title = "Boxplot der YouTube Likes", y = "YouTube Likes") +
  scale_y_continuous(labels = label_number(big.mark = ",", decimal.mark = "."))

ggplot(spotify_2024, aes(y = TikTok.Views)) +
  geom_boxplot(fill = "#1DB954") +
  labs(title = "Boxplot der TikTok Views", y = "TikTok Views") +
  scale_y_continuous(labels = label_number(big.mark = ",", decimal.mark = "."))

ggplot(spotify_2024, aes(y = TikTok.Likes)) +
  geom_boxplot(fill = "#1DB954") +
  labs(title = "Boxplot der TikTok Likes", y = "TikTok Likes") +
  scale_y_continuous(labels = label_number(big.mark = ",", decimal.mark = "."))
```

Die dargestellten Boxplots veranschaulichen die Daten der Variablen und stellen vor allem Ausreißer dar.


<br>
<br>
**<u>Dichteplot</u>**  
Die folgenden Dichteplots veranschaulichen die Verteilung der Werte einer Variable im Datensatz.
```{r}
ggplot(spotify_2024, aes(x = Spotify.Streams)) +
  geom_density(fill = "#1DB954", alpha = 0.5) +
  labs(title = "Dichteplot der Spotify Streams", x = "Spotify Streams", y = "Dichte") +
  scale_x_log10(labels = label_number(big.mark = ",", decimal.mark = ".")) +
  scale_y_continuous(labels = label_number(big.mark = ",", decimal.mark = "."))

ggplot(spotify_2024, aes(x = danceability)) +
geom_density(fill = "#1DB954", alpha = 0.5) +
labs(title = "Dichteplot der Danceability", x = "Danceability", y = "Dichte") +
scale_x_log10(labels = label_number(big.mark = ",", decimal.mark = ".")) +
scale_y_continuous(labels = label_number(accuracy = 0.00001, big.mark = ",", decimal.mark = "."))

ggplot(spotify_2024, aes(x = energy)) +
geom_density(fill = "#1DB954", alpha = 0.5) +
labs(title = "Dichteplot der Energy", x = "Energy", y = "Dichte") +
scale_x_log10(labels = label_number(big.mark = ",", decimal.mark = ".")) +
scale_y_continuous(labels = label_number(accuracy = 0.00001, big.mark = ",", decimal.mark = "."))

ggplot(spotify_2024, aes(x = Spotify.Popularity)) +
geom_density(fill = "#1DB954", alpha = 0.5) +
labs(title = "Dichteplot der Spotify Popularity", x = "Spotify Popularity", y = "Dichte") +
scale_x_log10(labels = label_number(big.mark = ",", decimal.mark = ".")) +
scale_y_continuous(labels = label_number(accuracy = 0.00001, big.mark = ",", decimal.mark = "."))

ggplot(spotify_2024, aes(x = duration_seconds)) +
geom_density(fill = "#1DB954", alpha = 0.5) +
labs(title = "Dichteplot der Songdauer", x = "Dauer in Sekunden", y = "Dichte") +
scale_x_log10(labels = label_number(big.mark = ",", decimal.mark = ".")) +
scale_y_continuous(labels = label_number(accuracy = 0.00001, big.mark = ",", decimal.mark = "."))

ggplot(spotify_2024, aes(x = Spotify.Playlist.Count)) +
geom_density(fill = "#1DB954", alpha = 0.5) +
labs(title = "Dichteplot der Spotify Playlist Count", x = "Spotify Playlist Count", y = "Dichte") +
scale_x_log10(labels = label_number(big.mark = ",", decimal.mark = ".")) +
scale_y_continuous(labels = label_number(accuracy = 0.00001, big.mark = ",", decimal.mark = "."))

ggplot(spotify_2024, aes(x = YouTube.Views)) +
geom_density(fill = "#1DB954", alpha = 0.5) +
labs(title = "Dichteplot der YouTube Views", x = "YouTube Views", y = "Dichte") +
scale_x_log10(labels = label_number(big.mark = ",", decimal.mark = ".")) +
scale_y_continuous(labels = label_number(accuracy = 0.00001, big.mark = ",", decimal.mark = "."))

ggplot(spotify_2024, aes(x = YouTube.Views)) +
geom_density(fill = "#1DB954", alpha = 0.5) +
labs(title = "Dichteplot der YouTube Views", x = "YouTube Views", y = "Dichte") +
scale_x_log10(labels = label_number(big.mark = ",", decimal.mark = ".")) +
scale_y_continuous(labels = label_number(big.mark = ",", decimal.mark = "."))

ggplot(spotify_2024, aes(x = YouTube.Likes)) +
geom_density(fill = "#1DB954", alpha = 0.5) +
labs(title = "Dichteplot der YouTube Likes", x = "YouTube Likes", y = "Dichte") +
scale_x_log10(labels = label_number(big.mark = ",", decimal.mark = ".")) +
scale_y_continuous(labels = label_number(accuracy = 0.00001, big.mark = ",", decimal.mark = "."))

ggplot(spotify_2024, aes(x = TikTok.Views)) +
geom_density(fill = "#1DB954", alpha = 0.5) +
labs(title = "Dichteplot der TikTok Views", x = "TikTok Views", y = "Dichte") +
scale_x_log10(labels = label_number(big.mark = ",", decimal.mark = ".")) +
scale_y_continuous(labels = label_number(accuracy = 0.00001, big.mark = ",", decimal.mark = "."))

ggplot(spotify_2024, aes(x = TikTok.Likes)) +
geom_density(fill = "#1DB954", alpha = 0.5) +
labs(title = "Dichteplot der TikTok Likes", x = "TikTok Likes", y = "Dichte") +
scale_x_log10(labels = label_number(big.mark = ",", decimal.mark = ".")) +
scale_y_continuous(labels = label_number(accuracy = 0.00001, big.mark = ",", decimal.mark = "."))
```

Die dargestellten Dichteplots veranschaulichen die geschätzte Verteilung der Werte der verschiedenen Variablen im Datensatz und ermöglichen die Identifizierung von Mustern.
  
<br>
<u> Korrelationsmatrix </u>  
  
Für die Untersuchung von Korrelationen im Datensatz werden alle numerischen Variablen untersucht und auch visualisiert.
  
```{r}

numeric_vars <- spotify_2024 %>%
  select(where(is.numeric))

correlation_matrix <- cor(numeric_vars, use = "pairwise.complete.obs")

ggcorrplot(correlation_matrix, 
           method = "square",  
           type = "full",              
           lab = TRUE,               
           lab_size = 1,                  
           colors = c("red", "white", "#1DB954"),
           tl.cex = 5,                   
           ggtheme = theme_minimal(),     
           title = "Korrelationsmatrix der Spotify-Daten") + 
   theme(plot.title = element_text(hjust = 0.5),
        axis.text.x = element_text(angle = 90, hjust = 1))
```

<br>
**<u>Interpretation der Korrelationen:</u>**  
Für die nachfolgende Analyse der aufgestellten Hypothesen gibt die Korrelationsmatrix bereits erste Hinweise, wie bestimmte Merkmale eines Songs dessen Popularität auf Spotify und anderen Plattformen beeinflussen können:  

<u>Spotify Streams und Playlist-Anzahl:</u>

* Die starke positive Korrelation (0,84) zeigt, dass Songs in vielen Playlists tendenziell mehr Streams haben.

<u>Spotify Streams und Playlist-Reichweite:</u>

* Eine moderate Korrelation (0,40) unterstützt die Annahme, dass größere Playlist-Reichweiten mit höheren Streams verbunden sind.

<u>Spotify Streams und YouTube Views/Likes:</u>

* Die moderaten positiven Korrelationen mit YouTube Views (0,56) und Likes (0,58) deuten darauf hin, dass erfolgreiche Songs auf Spotify auch auf anderen Plattformen wie YouTube erfolgreich sind.

<u>Spotify Streams und Spotify Popularity:</u>

* Es besteht eine moderate Korrelation (0,35) zwischen Streams und der Popularitätsbewertung auf Spotify, was darauf hindeutet, dass beliebte Songs in der Regel auch höhere Streaming-Zahlen haben.

<u>Spotify Streams und Shazam Counts:</u>

* Die starke Korrelation (0,65) bestätigt, dass Songs mit vielen Shazam-Suchen auch hohe Streaming-Zahlen haben.

<u>Tanzbarkeit und Spotify Streams:</u>

* Die negative Korrelation (-0,14) zwischen Tanzbarkeit und Spotify Streams deutet darauf hin, dass tanzbare Songs tendenziell unbeliebter sind.

<u>Explicit Content und Spotify Streams:</u>

* Die negative Korrelation (-0,15) legt nahe, dass Songs ohne expliziten Inhalt tendenziell mehr Streams haben.

<u>Energie und Tanzbarkeit:</u>

* Die schwach positive Korrelation (0,09) zwischen Energie und Tanzbarkeit ist relevant für die Hypothese, dass tanzbare Songs tendenziell energetischer sind.

<br>

# Analyseteil
Im Analyseteil werden die aufgestellten Hypothesen, die möglicherweise Einfluss auf die Popularität eines Spotify Songs haben, untersucht.

## Hypothese: Songs mit höherer Tanzbarkeit sind tendenziell beliebter

Als Maß der Beliebtheit eines Songs wird für diese Analyse die Variable 'Spotify.Popularity' verwendet. Zunächst schauen wir uns die Korrelation dieser Variable zur Variable 'Danceability' an und erstellen ein Streudiagramm zur veranschaulichung.
```{r}
correlation_dance_popularity <- cor(spotify_2024$danceability, spotify_2024$Spotify.Popularity, use = "complete.obs")
print(correlation_dance_popularity)

plot(spotify_2024$danceability, spotify_2024$Spotify.Popularity, 
     xlab = "Danceability", 
     ylab = "Spotify Popularity", 
     main = "Streudiagramm: Danceability vs Popularity",
     col = "#1DB954")
```

Wie bereits in der Explorativen Datenanalyse herausgestellt, stellt das Streudiagramm eine gleichmäßige Verteilung dar. Die Korrelation ist negativ und eher schwach. Dies bedeutet, dass tendenziiell Songs mit höherer Tanzbarkeit weniger beliebt sind.

Zur weiteren Analyse wird nun eine Regressionsanalyse durchgeführt, um den Einfluss der 'danceability' auf die 'Spotify.Popularity' zu bewerten.
```{r}
lm_model <- lm(Spotify.Popularity ~ danceability, data = spotify_2024)
summary(lm_model)
```

<u>Interpretation der Regressionsanalyse:</u>  
Das Ergebnis der Regressionsanalyse zeigt, dass es einen signifikanten negativen Zusammenhang zwischen der Tanzbarkeit eines Songs und dessen Beliebtheit auf Spotify gibt.

Die geschätzten Werte aus dem Modell sind:

* Intercept (Schnittpunkt): 76.316 Spotify.Popularity
* Koeffizient für danceability: -0.11949 Spotify.Popularity pro Einheit 'danceability'

Dies bedeutet, dass ein Song mit einer Tanzbarkeit von 0 eine durchschnittliche Beliebtheit von etwa 76,3 erreichen würde. Eine Erhöhung der Tanzbarkeit um ein Punkt würde mit einem Rückgang der Beliebtheit um etwa 0,1194 einhergehen.
Dies bedeutet, dass Songs mit höherer Tanzbarkeit im Durchschnitt eine niedrigere Beliebtheit aufweisen.

Somit wird die Hypothese, dass Songs mit höherer Tanzbarkeit beliebter sind, durch die Analyse widerlegt. Die Ergebnisse deuten vielmehr darauf hin, dass eine höhere Tanzbarkeit mit einer geringeren Beliebtheit verbunden ist.

## Hypothese: Kürzere Songs haben höhere Streaming-Zahlen als längere Songs
In diesem Schritt wird eine neue Variable Song_Laenge erstellt, die die Songs in zwei Kategorien unterteilt: "Kurz (<3 Min)" für Songs, die weniger als 180 Sekunden dauern, und "Lang (>=3 Min)" für Songs, die 180 Sekunden oder länger sind. Dies ermöglicht es, die Daten nach der Songlänge zu gruppieren und die durchschnittlichen Streaming-Zahlen innerhalb jeder Gruppe zu berechnen.
```{r}
spotify_2024$Song_Laenge <- ifelse(spotify_2024$duration_seconds < 180, "Kurz (<3 Min)", "Lang (>=3 Min)")
```

Im nächsten Schritt werden die durchschnittlichen Streaming-Zahlen (Spotify.Streams) für jede Kategorie der Songlänge berechnet. Die Funktion aggregate() gruppiert die Daten nach der neuen Variable Song_Laenge und berechnet den Mittelwert der Streaming-Zahlen für jede Gruppe.  
Dies hilft, einen ersten Überblick über die Unterschiede in den Streaming-Zahlen zwischen kurzen und langen Songs zu erhalten.
```{r}
summary_stats <- aggregate(Spotify.Streams ~ Song_Laenge, data = spotify_2024, FUN = mean)
print(summary_stats)
```

Um die Verteilung der Streaming-Zahlen für kurze und lange Songs zu visualisieren wird ein Boxplot erstellt.  
Der Boxplot zeigt die mittleren Werte, Quartile und mögliche Ausreißer der Streaming-Zahlen in beiden Kategorien. Dies ermöglicht eine schnelle visuelle Beurteilung, ob es Unterschiede in den Streaming-Zahlen zwischen kurzen und langen Songs gibt.
```{r}
boxplot(Spotify.Streams ~ Song_Laenge, data = spotify_2024,
        main = "Streaming-Zahlen nach Songlänge",
        xlab = "Songlänge",
        ylab = "Spotify-Streams",
        col = c("#1DB954", "#1DB954"))
```

Zuletzt folgt der t-test, um zu testen, ob der Unterschied in den durchschnittlichen Streaming-Zahlen zwischen kurzen und langen Songs statistisch signifikant ist.  
Die Nullhypothese (H0) lautet, dass es keinen Unterschied zwischen den beiden Gruppen gibt, während die Alternativhypothese (H1) besagt, dass kürzere Songs höhere Streaming-Zahlen haben als längere Songs.  
Der p-Wert des t-Tests zeigt an, ob der beobachtete Unterschied statistisch signifikant ist.
```{r}
t_test_result <- t.test(Spotify.Streams ~ Song_Laenge, data = spotify_2024)
print(t_test_result)
```
Das Ergebnis des t-Tests zeigt, dass es einen signifikanten Unterschied in den durchschnittlichen Spotify-Streams zwischen kürzeren und längeren Songs gibt.

Die durchschnittlichen Streams in den Testgruppen betragen:

* Gruppe "Kurz (<3 Min)": 453.956.522 Streams
* Gruppe "Lang (>=3 Min)": 616.125.448 Streams

Diese Ergebnisse zeigen, dass längere Songs im Durchschnitt etwa 162 Millionen mehr Streams haben als kürzere Songs. Dies deutet darauf hin, dass längere Songs tendenziell mehr gestreamt werden.

Der p-Wert von 0,0000002167 liegt deutlich unter dem Signifikanzniveau von 0,05, was darauf hinweist, dass der beobachtete Unterschied zwischen den beiden Gruppen statistisch signifikant ist und nicht durch Zufall erklärt werden kann.

Das 95%-Konfidenzintervall für den Unterschied der Mittelwerte reicht von -222.901.053 bis -101.436.799. Da dieses Intervall vollständig unter 0 liegt, bestätigt es, dass längere Songs konsistent mehr gestreamt werden. Selbst im ungünstigsten Fall haben längere Songs signifikant höhere Streaming-Zahlen als kürzere Songs.

Somit wird die Hypothese, dass kürzere Songs mehr Streaming-Zahlen haben als längere Songs, durch die Analyse widerlegt. Stattdessen deuten die Ergebnisse darauf hin, dass längere Songs tendenziell mehr Streams erzielen.

## Hypothese: Songs die in vielen Playlists vorhanden sind haben mehr Streams
In diesem Schritt wird die Korrelation zwischen Spotify.Playlist.Count und Spotify.Streams betrachtet, um zu sehen, wie stark die beiden Variablen miteinander korrelieren.
```{r}
correlation <- cor(spotify_2024$Spotify.Playlist.Count, spotify_2024$Spotify.Streams, use = "complete.obs")
cat("Korrelation zwischen Spotify.Playlist.Count und Spotify.Streams:", correlation, "\n")
```
In diesem Schritt wird ein Streudiagramm erstellt, um den Zusammenhang zwischen Spotify.Playlist.Count und Spotify.Streams visuell zu untersuchen. Um mögliche Trends, Ausreißer oder nicht-lineare Beziehungen zu erkennen.
```{r}
ggplot(spotify_2024, aes(x = Spotify.Playlist.Count, y = Spotify.Streams)) +
  geom_point(alpha = 0.6, color = "#1DB954") +
  geom_smooth(method = "lm", color = "black", se = FALSE) + 
  labs(title = "Zusammenhang zwischen Anzahl der Spotify-Playlists und Streams",
       x = "Anzahl der Spotify-Playlists",
       y = "Anzahl der Spotify-Streams") +
  scale_y_continuous(labels = scales::comma) +
  scale_x_continuous(labels = scales::comma) + 
  theme_minimal()
```
Die Korrelation beträgt 0.835 und ist stark positiv. Das bedeutet, dass Songs, die in mehr Playlists enthalten sind, tendenziell auch mehr Streams haben.

Nun wird eine Regressionsanalyse durchgeführt, um zu überprüfen, ob Spotify.Playlist.Count ein signifikanter Prädiktor für Spotify.Streams ist.
```{r}
model <- lm(Spotify.Streams ~ Spotify.Playlist.Count, data = spotify_2024)
summary(model)
```
<u>Interpretation der Regressionsanalyse:</u>  
Das Ergebnis der Regressionsanalyse zeigt, dass es einen signifikanten positiven Zusammenhang zwischen der Anzahl der Playlists, in denen ein Song enthalten ist, und der Anzahl der Spotify-Streams gibt.

Die geschätzten Werte für die Anzahl der Spotify-Streams basierend auf der Anzahl der Playlists sind:

* Intercept (Schnittpunkt): 75.200.000 Streams
* Koeffizient für Spotify.Playlist.Count: 5.380 Streams pro zusätzlicher Playlist

Das bedeutet, dass ein Song, der in keiner Playlist enthalten ist, im Durchschnitt etwa 75,2 Millionen Streams hat. Für jede zusätzliche Playlist, in der ein Song enthalten ist, steigen die Spotify-Streams durchschnittlich um 5.380 Streams. Dieser Zusammenhang ist statistisch hochsignifikant.

Somit wird die Hypothese, dass Songs, die in mehr Spotify-Playlists enthalten sind, mehr Streams haben, durch die Analyse stark gestützt.

## Hypothese: Songs mit expliziten Inhalten haben mehr Streams als Songs ohne expliziten Inhalt
In diesem Schritt werden die durchschnittlichen Spotify-Streams für Songs mit und ohne expliziten Inhalt berechnet:
```{r}
mean_streams_explicit <- mean(spotify_2024$Spotify.Streams[spotify_2024$Explicit.Track == 1])
mean_streams_non_explicit <- mean(spotify_2024$Spotify.Streams[spotify_2024$Explicit.Track == 0])

cat("Durchschnittliche Streams für explizite Songs:", mean_streams_explicit, "\n")
cat("Durchschnittliche Streams für nicht explizite Songs:", mean_streams_non_explicit, "\n")
```
Ein Boxplot wird erstellt, um die Verteilung der Spotify-Streams für Songs mit und ohne expliziten Inhalt visuell zu vergleichen:
```{r}

ggplot(spotify_2024, aes(x = factor(Explicit.Track), y = Spotify.Streams, fill = factor(Explicit.Track))) +
  geom_boxplot() +
  scale_y_log10() +
  labs(title = "Verteilung der Spotify Streams nach Explizit-Status",
       x = "Explizit-Status (0 = Nicht explizit, 1 = Explizit)",
       y = "Spotify Streams",
       fill = "Explizit-Status") +
       scale_fill_manual(values = c("0" = "#1DB954", "1" = "#FF5733")) +
       theme_minimal()
```

Ein t-Test wird durchgeführt, um zu prüfen, ob der Unterschied in den durchschnittlichen Streams zwischen den beiden Gruppen statistisch signifikant ist:
```{r}
t_test_result <- t.test(Spotify.Streams ~ Explicit.Track, data = spotify_2024, var.equal = FALSE)
print(t_test_result)
```
<u>Interpretation des t-Tests:</u>  
Das Ergebnis des t-Tests zeigt, dass es einen signifikanten Unterschied in den durchschnittlichen Spotify-Streams zwischen Songs mit und ohne expliziten Inhalt gibt.
Die durchschnittlichen Streams in den Testgruppen betragen:

* Gruppe 0 (ohne expliziten Inhalt): 615.072.556 Streams
* Gruppe 1 (expliziter Inhalt): 485.681.294 Streams

Das zeigt, dass Songs ohne expliziten Inhalt durchschnittlich etwa 129 Millionen mehr Streams haben als Songs mit expliziten Inhalte und deutet darauf hin, dass explizite Inhalte weniger gestreamt werden.

Somit ist die Hypothese, dass Songs mit expliziten Inhalten mehr Streams haben als solche ohne expliziten Inhalt, widerlegt.

## Hypothese: Beliebte Songs auf Spotify sind auch auf anderen Plattformen erfolgreich
Als Maß des Erfolges eines Songs werden nun die Variablen 'YouTube.Views' und 'TikTok.Views' verwendet.
Zuerst werden Streudiagramme dargestellt, um einen ersten Eindruck von möglichen Korrelationen zu gewinnen.
```{r}
ggplot(spotify_2024, aes(x = Spotify.Popularity, y = YouTube.Views)) +
  geom_point(alpha = 0.5, color = "#1DB954") +
  labs(title = "Spotify Popularity vs YouTube Views",
       x = "Spotify Popularity", 
       y = "YouTube Views") +
  scale_y_continuous(labels = scales::label_number(big.mark = ","))

ggplot(spotify_2024, aes(x = Spotify.Popularity, y = TikTok.Views)) +
  geom_point(alpha = 0.5, color = "#1DB954") +
  labs(title = "Spotify Popularity vs TikTok Views",
       x = "Spotify Popularity", 
       y = "TikTok Views") +
  scale_y_continuous(labels = scales::label_number(big.mark = ","))
```

Ob ein Zusammenhang besteht, können wir feststellen, indem wir die Korrelation zwischen 'Spotify.Popularity' und den anderen Variablen berechnen.
```{r}
correlation_youtube <- cor(spotify_2024$Spotify.Popularity, spotify_2024$YouTube.Views, use = "complete.obs")
correlation_tiktok <- cor(spotify_2024$Spotify.Popularity, spotify_2024$TikTok.Views, use = "complete.obs")

correlation_youtube
correlation_tiktok
```
<u>Interpretation der Korrelationen:</u>

* Spotify.Popularity und YouTube.Views (Korrelationskoeffizient: 0.0928):
    * Es gibt eine sehr schwache positive Beziehung zwischen der Popularität eines Songs auf Spotify und der Anzahl der Aufrufe auf YouTube. Der niedrige Korrelationskoeffizient deutet darauf hin, dass die Popularität auf Spotify nur einen minimalen Einfluss auf die Aufrufe auf YouTube hat und diese Beziehung nicht signifikant ist.
* Spotify.Popularity und TikTok.Views (Korrelationskoeffizient: 0.0198):
    * Es gibt nahezu keine lineare Beziehung zwischen der Popularität eines Songs auf Spotify und der Anzahl der Views auf TikTok. Der extrem niedrige Korrelationskoeffizient zeigt, dass der Erfolg auf Spotify keinen signifikanten Einfluss auf den Erfolg auf TikTok hat.

<br>
<br>
**<u>Lineare Regression Spotify.Popularity & YouTube.Views</u>**
```{r}
model_youtube <- lm(YouTube.Views ~ Spotify.Popularity, data = spotify_2024)
summary(model_youtube)

```
<u>Interpretation des Ergebnisses:</u>  

* Intercept (Schnittpunkt): 90.748.379 YouTube Views
* Koeffizient für Spotify.Popularity: steigt um 3.653.929 YouTube Views

Dies bedeutet, dass ein Song mit einer Popularität von 0 auf Spotify durchschnittlich etwa 90.7 Millionen YouTube-Aufrufe hätte. Bei einer Erhöhung der Spotify-Popularität um 1 Punkt würde die Anzahl der YouTube-Aufrufe um etwa 3.65 Millionen steigen.   

Es besteht eine schwache, aber signifikante Beziehung zwischen der Popularität auf Spotify und den YouTube-Aufrufen. Die Popularität auf Spotify ist jedoch kein starker Prädiktor für die Aufrufzahlen auf YouTube.
  
<br>
<br>
**<u>Lineare Regression Spotify.Popularity & TikTok Views</u>**
```{r}
model_tiktok <- lm(TikTok.Views ~ Spotify.Popularity, data = spotify_2024)
summary(model_tiktok)
```
<u>Interpretation des Ergebnisses:</u>

* Intercept (Schnittpunkt): 718.998.435 Views
* Koeffizient für Spotify.Popularity: 3.013.265 Views pro Punkt Spotify.Popularity

Dies bedeutet, dass ein Song mit einer Popularität von 0 auf Spotify durchschnittlich etwa 719 Millionen TikTok-Views hätte. Bei einer Erhöhung der Spotify-Popularität um 1 Punkt würden die TikTok-Views um etwa 3.01 Millionen steigen.

Es gibt keinen statistisch signifikanten Zusammenhang zwischen der Popularität auf Spotify und den TikTok-Views. Die Popularität auf Spotify hier ebenfalls kein verlässlicher Prädiktor für den Erfolg auf TikTok.

## Hypothese: Songs mit höherem BPM sind tendenziell beliebter auf Streaming-Plattformen

Ein Streudiagramm wird erstellt, um die Beziehung zwischen dem BPM eines Songs und seiner Beliebtheit auf Spotify zu visualisieren. Jeder Datenpunkt im Diagramm stellt einen Song dar.
```{r}

ggplot(spotify_2024, aes(x = bpm, y = Spotify.Popularity)) +
  geom_point(alpha = 0.5, color = "#1DB954") +
  labs(title = "BPM vs. Spotify Popularity", x = "BPM", y = "Spotify Popularity") +
  theme_minimal()
```

Ein weiteres Streudiagramm wird erstellt, um die Beziehung zwischen BPM und den Streamingzahlen auf Spotify zu untersuchen.
```{r}
ggplot(spotify_2024, aes(x = bpm, y = Spotify.Streams)) +
  geom_point(alpha = 0.5, color = "#1DB954") +
  scale_y_continuous(labels = scales::label_number(scale_cut = scales::cut_short_scale())) +
  labs(title = "BPM vs. Spotify Streams", x = "BPM", y = "Spotify Streams") +
  theme_minimal()

```

Hier wird die Korrelation zwischen BPM und Spotify.Popularity sowie zwischen BPM und Spotify.Streams berechnet.
```{r}
cor_bpm_popularity <- cor(spotify_2024$bpm, spotify_2024$Spotify.Popularity, use = "complete.obs")
print(paste("Korrelationskoeffizient zwischen BPM und Spotify Popularity:", cor_bpm_popularity))

cor_bpm_streams <- cor(spotify_2024$bpm, spotify_2024$Spotify.Streams, use = "complete.obs")
print(paste("Korrelationskoeffizient zwischen BPM und Spotify Streams:", cor_bpm_streams))
```
<u>Interpretation der Korrelationen:</u>

* BPM und Spotify Popularity (Korrelationskoeffizient: 0.0119):
    * Es gibt nahezu keine Beziehung zwischen dem BPM (Beats Per Minute) und der Popularität eines Songs auf Spotify. Der sehr niedrige Korrelationskoeffizient deutet darauf hin, dass der BPM keinen signifikanten Einfluss auf die Popularität hat.
* BPM und Spotify Streams (Korrelationskoeffizient: -0.0143):
    * Auch hier zeigt der Korrelationskoeffizient eine sehr schwache negative Beziehung zwischen BPM und der Anzahl der Streams eines Songs auf Spotify. Dies bedeutet, dass der BPM nahezu keinen Einfluss auf die Anzahl der Streams hat.


Eine Regressionsanalyse wird durchgeführt, um zu untersuchen, wie gut der BPM eines Songs die Beliebtheit auf Spotify vorhersagt. Das Modell liefert eine Gleichung, die den Zusammenhang zwischen BPM und Beliebtheit beschreibt.
```{r}
lm_popularity <- lm(Spotify.Popularity ~ bpm, data = spotify_2024)
summary(lm_popularity)
```
<u>Interpretation des Ergebnisses:</u>

Die geschätzten Werte aus der Analyse sind:

* Intercept (Schnittpunkt): 67.573 Spotify Beliebtheit
* Koeffizient für BPM: 0.005 Beliebtheit pro BPM

Dies bedeutet, dass ein Song mit einem BPM von 0 im Durchschnitt eine Beliebtheit von etwa 67.573 auf Spotify hätte. Für jeden zusätzlichen BPM steigt der Beliebtheitswert des Songs um etwa 0.005 Einheiten.
<br>
Insgesamt wird die Hypothese, dass ein höherer BPM mit einer höheren Beliebtheit auf Spotify verbunden ist, durch diese Analyse nicht unterstützt. Der geringe R²-Wert und die fehlende Signifikanz des Koeffizienten legen nahe, dass BPM keinen signifikanten Einfluss auf die Beliebtheit eines Songs auf Spotify hat.

Eine weitere Regressionanalyse wird durchgeführt, um zu untersuchen, wie gut BPM die Anzahl der Streams vorhersagen kann. Auch hier liefert das Modell eine Gleichung und bewertet den Einfluss von BPM auf die Streamingzahlen.
```{r}
lm_streams <- lm(Spotify.Streams ~ bpm, data = spotify_2024)
summary(lm_streams)
```
<u>Interpretation des Ergebnisses:</u>

* Intercept (Schnittpunkt): 383.000.000 Streams
* Koeffizient für BPM: -275.000 Streams

Dies bedeutet, dass ein Song mit einem BPM von 0 im Durchschnitt etwa 383 Millionen Streams hätte.  
Für jeden zusätzlichen BPM sinkt die Anzahl der Spotify-Streams im Durchschnitt um etwa 275.000.

Insgesamt wird die Hypothese, dass ein höherer BPM mit einer höheren Anzahl an Spotify-Streams verbunden ist, durch diese Analyse nicht unterstützt. Der geringe R²-Wert und die fehlende Signifikanz des Koeffizienten legen nahe, dass der BPM eines Songs keinen wesentlichen Einfluss auf die Anzahl der Spotify-Streams hat.

## Hypothese: Songs mit höherer Tanzbarkeit haben tendenziell auch höhere Energie-Werte.

Um den Zusammenhang zu analysieren benötigen wir nun die Variablen 'danceability' und 'energy'.  
Dazu schauen wir uns die Korrelation und ein Streudiagramm an.
```{r}
correlation_dance_energy <- cor(spotify_2024$danceability, spotify_2024$energy, use = "complete.obs")
print(correlation_dance_energy)

plot(spotify_2024$danceability, spotify_2024$energy, 
     xlab = "Danceability", 
     ylab = "Energy", 
     main = "Streudiagramm: Danceability vs Energy",
     col = "#1DB954")
```

Die Korrelation von 0.0865 ist sehr schwach positiv. Dies deutet darauf hin, dass es nur einen minimalen Zusammenhang zwischen der 'Danceability' und 'Energy' eines Songs gibt. Dies veranschaulicht auch das Streudiagramm.

Zur tiefergehenden Analyse wird nun eine lineare Regression durchgeführt.
```{r}
lm_model <- lm(energy ~ danceability, data = spotify_2024)
summary(lm_model)
```
<u>Interpretation des Ergebnisses:</u>

* Intercept (Schnittpunkt): 56.198 Energie-Wert
* Koeffizient für Tanzbarkeit: 0.09667 Energie-Wert

Dies bedeutet, dass ein Song mit einer Tanzbarkeit von 0 im Durchschnitt einen Energie-Wert von etwa 56,2 aufweist. Für jede Erhöhung der Tanzbarkeit um eine Einheit steigt der Energie-Wert um etwa 0,097.

Der p-Wert für den Koeffizienten der Tanzbarkeit beträgt 0.0174, was unter dem Signifikanzniveau von 0.05 liegt. Dies deutet darauf hin, dass der Zusammenhang zwischen Tanzbarkeit und Energie statistisch signifikant ist.

Insgesamt wird die Hypothese, dass Songs mit höherer Tanzbarkeit tendenziell auch höhere Energie-Werte haben, durch diese Analyse unterstützt, jedoch mit dem Hinweis, dass der Einfluss relativ gering ist, wie der niedrige R²-Wert zeigt.

<br>

# Finden von Modellen

In diesem Abschnitt werden verschiedene Modelle untersucht, um zukünftige Ergebnisse basierend auf bestehenden Daten vorherzusagen. Sowohl Entscheidungsbäume als auch neuronale Netze werden verwendet, um die Variable Spotify.Popularity vorherzusagen.

## Datenaufbereitung

Zunächst werden aus dem umfangreichen Datensatz die Variablen extrahiert, die für die Modelle relevant sind. Der Fokus liegt dabei auf musikalischen Eigenschaften sowie weiteren Variablen, die potenziell Einfluss auf die Popularität von Spotify-Songs haben könnten und bei Veröffentlichung des Songs zugänglich sind. Diese Auswahl ermöglicht eine gezielte Modellierung der Vorhersage, indem nur die Merkmale berücksichtigt werden, die eine hohe Bedeutung für die Klassifikation der Popularität aufweisen.

```{r}
ml_data <- spotify_2024[c("duration_ms", "bpm", "energy", "Spotify.Popularity", "Explicit.Track", "key", "mode", "valence", "acousticness", "instrumentalness", "liveness", "speechiness")]

head(ml_data)
```

Die Zielvariable für die Modelle ist 'Spotify.Popularity', deren Werte derzeit zwischen 4 und 96 liegen. Um die Analyse zu vereinfachen, wird diese Variable in zwei Kategorien unterteilt: „Populär“ und „NichtPopulär“. Dabei dient ein Schwellwert von 50% als Trennlinie. Songs, die eine Popularität von 50 oder höher erreichen, werden als „Populär“ eingestuft, während Songs mit einem Wert darunter in die Kategorie „NichtPopulär“ fallen.

```{r}
# Schwellwert festlegen (in diesem Fall 50%)
schwellwert <- quantile(ml_data$Spotify.Popularity, 0.5)

# Kategorisieren in Populär und NichtPopulär
ml_data$popular <- ifelse(ml_data$Spotify.Popularity >= schwellwert, "Populär", "NichtPopulär")

# Umwandlung in Faktor
ml_data$popular <- as.factor(ml_data$popular)

# Löschen der nun nicht mehr benötigten Spotify.Popularity
ml_data$Spotify.Popularity <- NULL
```

Der finale Datensatz sieht dann wie folgt aus:

```{r}
head(ml_data)
```


## Entscheidungsbaum

Ein Entscheidungsbaum ist ein grafisches Modell, das zur Klassifizierung von Daten eingesetzt wird. Er stellt Entscheidungsregeln in einer baumartigen Struktur dar und ermöglicht eine nachvollziehbare Aufteilung der Daten basierend auf spezifischen Merkmalen. Der Baum wird so konstruiert, dass in jedem Knoten eine Regel angewendet wird, um die Daten in zwei oder mehr Gruppen aufzuteilen, wobei jede Aufteilung darauf abzielt, die Vorhersage der Zielvariable zu verbessern. Durch die visuelle Darstellung lassen sich die Entscheidungsprozesse leicht nachvollziehen, was zu einer klaren und interpretierbaren Klassifizierung führt.

### Modell erstellen

Um das Modell zu trainieren und anschließend zu evaluieren, wird der Datensatz in Trainings- und Testdaten aufgeteilt. Dabei werden 80% der Daten für das Training des Modells verwendet, während die restlichen 20% als Testdaten dienen, um die Leistung des Modells zu bewerten.

```{r}
inTrain <- createDataPartition(y = ml_data$popular, p = .9, list = FALSE)

ml_data_train <- ml_data[inTrain, ]
ml_data_test <- ml_data[-inTrain, ]
```

Jetzt wird das Entscheidungsbaum-Modell mit den Trainingsdaten erstellt.

```{r}
tree_model <- rpart(popular ~ ., 
                    data = ml_data_train, 
                    method = "class", 
                    control = rpart.control(maxdepth = 5, minsplit = 20))

tree_model
```

Um die Struktur des Entscheidungsbaums besser zu verstehen, wird die rpart.plot-Bibliothek verwendet. Diese Bibliothek bietet die Möglichkeit, die Entscheidungslogik des Modells klar und anschaulich darzustellen.

```{r}
rpart.plot(tree_model, extra = 102)
```

Der Entscheidungsbaum zur Vorhersage der Spotify-Popularität zeigt, wie verschiedene Merkmale die Wahrscheinlichkeit beeinflussen, ob ein Song populär oder nicht populär ist. Die Popularität eines Songs wird dabei stark von der 'Speechiness', 'Liveness', 'Energy' und dem 'Valence' abhängt.

Im Detail verdeutlicht der Baum, dass Songs mit einer Speechiness von mehr als 5,3 zunächst weiter nach höheren Werten (über 9,4) unterteilt werden. Diese Songs weisen eine deutlich erhöhte Wahrscheinlichkeit auf, nicht populär zu sein, insbesondere wenn ihre Energy unter 90 liegt. Wenn jedoch ein Song eine hohe Valence von über 23 hat, steigt die Wahrscheinlichkeit, dass er populär ist.

Darüber hinaus zeigt der Baum, dass Songs mit einer Liveness unter 11 tendenziell populärer sind, insbesondere wenn sie in vielen Playlists vertreten sind. Für Songs mit einer Liveness von 10 oder mehr sinkt die Wahrscheinlichkeit für eine hohe Popularität.

Zusammenfassend betont der Baum die Wichtigkeit der Variable 'Speechiness' und 'Liveness' als Hauptmerkmale, während Werte in 'Energy' und 'Valence' einen nicht ganz so großen Einfluss nehmen.

### Modell testen

In diesem Abschnitt wird das trainierte Entscheidungsbaum-Modell auf den Testdaten angewendet, um die Vorhersagegenauigkeit zu bewerten. Die Vorhersagen werden mit den tatsächlichen Werten verglichen, um die Leistungsfähigkeit des Modells zu analysieren. Zudem wird eine ROC-Kurve erstellt, um die Genauigkeit des Modells zu visualisieren.

```{r}
predictions <- predict(tree_model, newdata = ml_data_test, type = "prob")

# Vorhersagewahrscheinlichkeit für die Klasse "Populär"
pred_prob <- predictions[, "Populär"]

# ROC-Kurve erstellen
roc_curve_tree <- roc(ml_data_test$popular, pred_prob)

# ROC-Kurve plotten
plot(roc_curve_tree, main = "ROC-Kurve für Entscheidungsbaum", col = "#1DB954", lwd = 2)
```

Die ROC-Kurve für den Entscheidungsbaum zeigt die Leistungsfähigkeit des Modells bei der Klassifizierung von Songs in „Populär“ und „NichtPopulär“.

Eine Kurve, die stark in die obere linke Ecke neigt, weist auf eine gute Klassifikation hin. Liegt die Kurve über der diagonalen Linie, bedeutet dies, dass das Modell besser als Zufall ist. Insgesamt bietet die ROC-Kurve eine nützliche Bewertung der Modellleistung.

Die ROC-Kurve zeigt, dass das Modell eine gute Fähigkeit hat, zwischen „Populär“ und „NichtPopulär“ zu unterscheiden. Insgesamt deutet die ROC-Kurve auf eine akzeptable Leistung des Modells hin, mit Potenzial für weitere Optimierungen.

## Neuronales Netz

In diesem Abschnitt wird ein neuronales Netz erstellt, um die Popularität von Songs vorherzusagen. Hierbei kommen die Merkmale 'Speechiness', 'Energy' und 'Valence' als Eingabefunktionen zum Einsatz. Das Modell wird mit den Trainingsdaten trainiert, wobei eine versteckte Schicht mit fünf Neuronen verwendet wird. Das Ergebnis wird als Klassifikation behandelt, um die Popularität in die Kategorien "Populär" und "NichtPopulär" einzuordnen

### Modell erstellen

Im Folgenden werden die genutzten Variablen zwischengespeichert.

```{r}
variables_nn <- c("popular", "speechiness", "energy", "valence")
```


Hier wird das neuronale Netz mithilfe der vorher abgegrenzten Trainingsdaten erstellt.

```{r}
nn <- neuralnet(popular ~ speechiness + energy + valence, 
                data = ml_data_train[variables_nn], 
                hidden = c(5), 
                linear.output = FALSE)

```

### Modell testen

Im Folgenden wird das neuronale Netz visualisiert, um die Struktur und die Verbindungen zwischen den Neuronen zu überprüfen.

```{r}
plot(nn, rep = "best")
```

Anschließend werden Vorhersagen für die Testdaten mithilfe des trainierten neuronalen Netzwerks erstellt. Hierbei werden ausschließlich die definierten Merkmale (ohne die Zielvariable 'popular') verwendet, um die Vorhersagewahrscheinlichkeiten für die Klasse "Populär" zu generieren.

```{r}
predicted <- neuralnet::compute(nn, ml_data_test[variables_nn[-1]]) # -1 um popular nicht zu nutzen 

predicted_probabilities <- predicted$net.result[, 1]

ml_data_test$popular <- as.factor(ml_data_test$popular)

roc_curve_nn <- roc(ml_data_test$popular, predicted_probabilities, levels = c("NichtPopulär", "Populär"))
```

Zur Erläuterung der Modellleistung wird auch hier die ROC Kurve des neuronalen Netzes erstellt. Diese wird mit der Kurve des Entscheidungsbaums verglichen.

```{r}

plot(roc_curve_nn, main = "ROC-Kurve für Neuronales Netz und Entscheidungsbaum", col = "blue", lwd = 2)
lines(roc_curve_tree, col = "#1DB954", lwd = 2)

legend("bottomright", legend = c("Neuronales Netz", "Entscheidungsbaum"), 
       col = c("blue", "#1DB954"), lwd = 2)

```

Beide ROC-Kurven weisen ähnliche allgemeine Muster auf, was auf eine vergleichbare Leistung der Modelle hinweist. Auffällig ist jedoch, dass die ROC-Kurve des Entscheidungsbaums eine glattere Form aufweist, während die ROC-Kurve des neuronalen Netzwerks mehrere Ecken zeigt.

Dies könnte darauf hindeuten, dass der Entscheidungsbaum in seinen Vorhersagen stabiler ist, während das neuronale Netzwerk möglicherweise empfindlicher auf kleine Datenänderungen reagiert. Die Ecken in der ROC-Kurve des neuronalen Netzwerks deuten darauf hin, dass es in bestimmten Bereichen stärkere Schwankungen in der Modellleistung gibt. Dies kann durch zusätzliche Optimierungsansätze verbessert werden.

Es ist wichtig zu erwähnen, dass diese Ausführung der Modelle nur eine grundlegende Einführung in das maschinelle Lernen bieten soll. Mit einem größeren Datensatz und weiteren ergänzenden Methoden können die Modelle stark optimiert und trainiert werden.

# Ergebnisinterpretation

Wie zu Beginn beschrieben, war das Ziel dieser Ausarbeitung, mögliche Einflussfaktoren auf die Beliebtheit eines Spotify-Songs zu identifizieren. Dazu wurden die aufgestellten Hypothesen individuell untersucht, was es ermöglichte, diese zu stützen oder zu widerlegen.

Die Ergebnisse zeigen, dass mehrere Faktoren die Beliebtheit eines Spotify-Songs beeinflussen können. Besonders relevant ist, ob ein Song expliziten Inhalt enthält oder nicht; dies scheint einen erheblichen Einfluss auf die Hörerzahlen zu haben. Darüber hinaus können die Länge eines Songs und die Danceability positive Effekte auf die Beliebtheit haben.

Im Gegensatz dazu erwiesen sich Faktoren wie die Beats per Minute (BPM) und die Popularität auf anderen Plattformen, wie TikTok oder YouTube, als weniger signifikant. Dies deutet darauf hin, dass Hörer möglicherweise andere Kriterien zur Bewertung von Songs heranziehen, die nicht direkt mit der Musikproduktion oder der Präsentation auf sozialen Medien verbunden sind.

Diese Erkenntnisse sind insbesondere für Künstler und Musikproduzenten von großer Bedeutung, da sie gezielte Strategien entwickeln können, um populäre Songs zu kreieren. Zudem könnte es für zukünftige Forschungen interessant sein, weitere Einflussfaktoren zu untersuchen, wie etwa die Vermarktungsstrategien oder die Rolle von Playlists, um ein umfassenderes Bild der Erfolgsfaktoren in der Musikindustrie zu erhalten.
