---
title: "Das neue Flatfile CSV-Format (ffcsv) der Datenbank GENESIS-Online"
output: 
  pdf_document:
    latex_engine: xelatex
header-includes:
  - \usepackage{fontspec}
  - \setmainfont{Statis Sans Book}
---



Lauffähiger R-Code zum Einlesen


```r
library(pacman)
pacman::p_load(dplyr, readr, tidyr, stringr, tibble)
```

### Das bleibt konstant:
- Datensatz-orientiertes Format zur automatisierten Weiterverarbeitung
- Spalten durch Semikolon separiert (de, en)
- Dezimalkomma je nach Sprache "," (de) oder "." (en)
- utf-8 Codierung
- Zellen können wertersetzende Qualitätskennzeichen enthalten wie "-", "x", ".", "/"
- Spalte(n) Qualitätskennzeichen mit "_q" Suffix beim Download an- oder abwählbar

# Beispiel 1
``Tabelle: 61111-0001``

**Verbraucherpreisindex: Deutschland, Jahre**

### Das bisherige ffcsv-Format
- Preisindex und Veränderungsrate in getrennten
- Statistikspezifische Spaltenüberschriften


```r
tabelle61111_0001alt <- utils::read.csv("61111-0001_de_flat.csv", 
                                        sep = ";", 
                                        dec = ",", 
                                        na.strings = c("-", "x", ".", "/"))

tabelle61111_0001alt %>%
  dplyr::select("Zeit", 
                "PREIS1__Verbraucherpreisindex__2020.100",
                "PREIS1__Verbraucherpreisindex__q",
                "Verbraucherpreisindex__CH0004",
                "Verbraucherpreisindex__CH0004__q") %>% 
  tibble::as_tibble() %>% 
  utils::tail(3)
```

```
## # A tibble: 3 × 5
##    Zeit PREIS1__Verbraucherpreisindex__2020.100 PREIS1__Verbraucherpr…¹ Verbraucherpreisinde…²
##   <int>                                   <dbl> <chr>                                    <dbl>
## 1  2021                                    103. e                                          3.1
## 2  2022                                    110. e                                          6.9
## 3  2023                                    117. e                                          5.9
## # ℹ abbreviated names: ¹​PREIS1__Verbraucherpreisindex__q, ²​Verbraucherpreisindex__CH0004
## # ℹ 1 more variable: Verbraucherpreisindex__CH0004__q <chr>
```

### Das neue ffcsv-Format
- Einheitliche englische Spaltenbeschriftungen für de/en
- Eine Wertespalte "value" unabhängig von der Statistik
- Eine Spalte "value_unit" für die Einheit
- Auslieferung als ZIP-Datei (eine ffcsv-Datei je ZIP-Archiv)
- unsortiert


```r
# Speichere den entpackten Inhalt in einen temporären Ordner
# Kann auch ein dauerhafter Ordner z.B. auf dem Laufwerk sein
extract_dir <- base::tempdir() 

utils::unzip("61111-0001_de_flat.zip",
             exdir = extract_dir)

# Extrahiere die Datei mit Endung _flat.csv aus dem entpackten Inhalt
extracted_file <- base::list.files(extract_dir,
                                   full.names = TRUE,
                                   pattern = "_flat.csv")

tabelle61111_0001neu <- readr::read_delim(file = extracted_file,
                                          delim = ";",
                                          show_col_types = FALSE,
                                          locale = readr::locale(decimal_mark = ",",
                                                                 grouping_mark = "."),
                                          name_repair = "minimal",
                                          col_types = "c",
                                          na = c("-", "x", ".", "/")) %>% 
                          dplyr::arrange(time, value_variable_label)

tabelle61111_0001neu %>%
  dplyr::select("time", 
                "value",
                "value_unit",
                "value_variable_code",
                "value_variable_label",
                "value_q") %>% 
  tibble::as_tibble() %>% 
  utils::tail(6)
```

```
## # A tibble: 6 × 6
##    time value value_unit value_variable_code value_variable_label  value_q
##   <dbl> <dbl> <chr>      <chr>               <chr>                 <chr>  
## 1  2021 103.  2020=100   PREIS1              Verbraucherpreisindex e      
## 2  2021   3.1 %          PREIS1              in                    e      
## 3  2022 110.  2020=100   PREIS1              Verbraucherpreisindex e      
## 4  2022   6.9 %          PREIS1              in                    e      
## 5  2023 117.  2020=100   PREIS1              Verbraucherpreisindex e      
## 6  2023   5.9 %          PREIS1              in                    e
```

```r
# Entferne die temporäre csv-Datei aus dem temporären Ordner
base::invisible(base::file.remove(extracted_file))
```


```r
tabelle61111_0001neu %>% 
  tidyr::pivot_wider(id_cols = "time", 
                     names_from = "value_unit", 
                     values_from = "value") %>% 
  tibble::as_tibble() %>% 
  utils::tail(3)
```

```
## # A tibble: 3 × 3
##    time `2020=100`   `%`
##   <dbl>      <dbl> <dbl>
## 1  2021       103.   3.1
## 2  2022       110.   6.9
## 3  2023       117.   5.9
```

# Beispiel 2
``Tabelle: 61111-0003``

**Verbraucherpreisindex: Deutschland, Jahre, Klassifikation der Verwendungszwecke des Individualkonsums (COICOP 2-5-Steller Hierarchie)**

### Das bisherige ffcsv-Format
- COICOP 2- bis 5-Steller gemischt in einer Spalte "2_Auspraegung_Code"
- 2- bis 5-Steller mussten über die Länge der Ausprägungscodes gefiltert werden


```r
tabelle61111_0003alt <- utils::read.csv("61111-0003_de_flat.csv", 
                                        sep = ";", 
                                        dec = ",", 
                                        na.strings = c("-", "x", ".", "/"),
                                        check.names = FALSE)

tabelle61111_0003alt %>%
  dplyr::select("Zeit", 
                "2_Merkmal_Code",
                "2_Merkmal_Label",
                "2_Auspraegung_Code",
                "2_Auspraegung_Label",
                "PREIS1__Verbraucherpreisindex__2020=100",
                "PREIS1__Verbraucherpreisindex__q") %>% 
  tibble::as_tibble() %>% 
  utils::tail(15)
```

```
## # A tibble: 15 × 7
##     Zeit `2_Merkmal_Code` `2_Merkmal_Label`         `2_Auspraegung_Code` `2_Auspraegung_Label`
##    <int> <chr>            <chr>                     <chr>                <chr>                
##  1  2023 CC13A5           Verwendungszwecke des In… CC13-1253            "    Versicherungsdi…
##  2  2023 CC13A5           Verwendungszwecke des In… CC13-12532           "      Dienstl. priv…
##  3  2023 CC13A5           Verwendungszwecke des In… CC13-1254            "    Versicherungsdi…
##  4  2023 CC13A5           Verwendungszwecke des In… CC13-12541           "      Versicherungs…
##  5  2023 CC13A5           Verwendungszwecke des In… CC13-12542           "      Versicherungs…
##  6  2023 CC13A5           Verwendungszwecke des In… CC13-1255            "    Andere Versiche…
##  7  2023 CC13A5           Verwendungszwecke des In… CC13-12550           "      Andere Versic…
##  8  2023 CC13A5           Verwendungszwecke des In… CC13-1262            "    Andere Finanzdi…
##  9  2023 CC13A5           Verwendungszwecke des In… CC13-12621           "      Bank- und Spa…
## 10  2023 CC13A5           Verwendungszwecke des In… CC13-12622           "      Gebühren für …
## 11  2023 CC13A5           Verwendungszwecke des In… CC13-1270            "    Andere Dienstle…
## 12  2023 CC13A5           Verwendungszwecke des In… CC13-12701           "      Verwaltungsge…
## 13  2023 CC13A5           Verwendungszwecke des In… CC13-12702           "      Rechtsberatun…
## 14  2023 CC13A5           Verwendungszwecke des In… CC13-12703           "      Bestattungsle…
## 15  2023 CC13A5           Verwendungszwecke des In… CC13-12704           "      Andere Gebühr…
## # ℹ 2 more variables: `PREIS1__Verbraucherpreisindex__2020=100` <dbl>,
## #   PREIS1__Verbraucherpreisindex__q <chr>
```

- Die Spalte "2_Auspraegung_Code" enthält gemischt 2- bis 5-Steller Codes, die über die Länge der Codes gefiltert werden mussten


```r
tabelle61111_0003alt %>% 
  dplyr::filter(Zeit == "2023") %>% 
  dplyr::group_by(`2_Merkmal_Code`) %>% 
  dplyr::summarise(value_count = n())
```

```
## # A tibble: 1 × 2
##   `2_Merkmal_Code` value_count
##   <chr>                  <int>
## 1 CC13A5                   385
```

### Das neue ffcsv-Format
- Über die Spalte "2_variable_code" werden die verschiedenen COICOP-Hierarchien filterbar
- Jede COICOP-Hierarchiestufe ist separat vollständig auswertbar


```r
# Speichere den entpackten Inhalt in einen temporären Ordner
# Kann auch ein dauerhafter Ordner z.B. auf dem Laufwerk sein
extract_dir <- base::tempdir() 

utils::unzip("61111-0003_de_flat.zip",
             exdir = extract_dir)

# Extrahiere die Datei mit Endung _flat.csv aus dem entpackten Inhalt
extracted_file <- base::list.files(extract_dir,
                                   full.names = TRUE,
                                   pattern = "_flat.csv")

tabelle61111_0003neu <- readr::read_delim(file = extracted_file,
                                          delim = ";",
                                          show_col_types = FALSE,
                                          locale = readr::locale(decimal_mark = ",",
                                                                 grouping_mark = "."),
                                          name_repair = "minimal",
                                          col_types = "c",
                                          na = c("-", "x", ".", "/")) %>% 
                          dplyr::arrange(time, `2_variable_attribute_code`)

# Entferne die temporäre csv-Datei aus dem temporären Ordner
base::invisible(base::file.remove(extracted_file))

tabelle61111_0003neu %>%
  dplyr::filter(stringr::str_detect(`2_variable_attribute_code`, "CC13-12")) %>% 
  dplyr::select("time", 
                "2_variable_code",
                "2_variable_label",
                "2_variable_attribute_code",
                "2_variable_attribute_label",
                "value") %>% 
  tibble::as_tibble() %>% 
  utils::tail(15)
```

```
## # A tibble: 15 × 6
##     time `2_variable_code` `2_variable_label`    2_variable_attribute…¹ 2_variable_attribute…²
##    <dbl> <chr>             <chr>                 <chr>                  <chr>                 
##  1  2023 CC13A4            Verwendungszwecke de… CC13-1254              Versicherungsdienstle…
##  2  2023 CC13A5            Verwendungszwecke de… CC13-12541             Versicherungsdienstl.…
##  3  2023 CC13A5            Verwendungszwecke de… CC13-12542             Versicherungsdienstle…
##  4  2023 CC13A4            Verwendungszwecke de… CC13-1255              Andere Versicherungsd…
##  5  2023 CC13A5            Verwendungszwecke de… CC13-12550             Andere Versicherungsd…
##  6  2023 CC13A3            Verwendungszwecke de… CC13-126               Finanzdienstleistunge…
##  7  2023 CC13A4            Verwendungszwecke de… CC13-1262              Andere Finanzdienstle…
##  8  2023 CC13A5            Verwendungszwecke de… CC13-12621             Bank- und Sparkasseng…
##  9  2023 CC13A5            Verwendungszwecke de… CC13-12622             Gebühren für Anlagebe…
## 10  2023 CC13A3            Verwendungszwecke de… CC13-127               Andere Dienstleistung…
## 11  2023 CC13A4            Verwendungszwecke de… CC13-1270              Andere Dienstleistung…
## 12  2023 CC13A5            Verwendungszwecke de… CC13-12701             Verwaltungsgebühren   
## 13  2023 CC13A5            Verwendungszwecke de… CC13-12702             Rechtsberatung, Recht…
## 14  2023 CC13A5            Verwendungszwecke de… CC13-12703             Bestattungsleistungen…
## 15  2023 CC13A5            Verwendungszwecke de… CC13-12704             Andere Gebühren und D…
## # ℹ abbreviated names: ¹​`2_variable_attribute_code`, ²​`2_variable_attribute_label`
## # ℹ 1 more variable: value <dbl>
```

- Über die Spalte "2_variable_code" werden die verschiedenen COICOP-Hierarchien filterbar


```r
tabelle61111_0003neu %>% 
  dplyr::filter(time == "2023") %>% 
  dplyr::group_by(`2_variable_code`) %>% 
  dplyr::summarise(value_count = n())
```

```
## # A tibble: 4 × 2
##   `2_variable_code` value_count
##   <chr>                   <int>
## 1 CC13A2                     12
## 2 CC13A3                     44
## 3 CC13A4                    110
## 4 CC13A5                    275
```


```r
tabelle61111_0003neu %>% 
  tidyr::pivot_wider(id_cols = c("2_variable_label", "2_variable_attribute_label"), 
                     names_from = "time", 
                     values_from = "value") %>% 
  dplyr::arrange(`2_variable_label`) %>% 
  tibble::as_tibble()
```

```
## # A tibble: 441 × 7
##    `2_variable_label`                2_variable_attribute…¹ `2019` `2020` `2021` `2022` `2023`
##    <chr>                             <chr>                   <dbl>  <dbl>  <dbl>  <dbl>  <dbl>
##  1 Verwendungszwecke des Individual… Nahrungsmittel und al…   97.9    100  103.   116    130. 
##  2 Verwendungszwecke des Individual… Alkoholische Getränke…   98.1    100  104.   108.   117. 
##  3 Verwendungszwecke des Individual… Bekleidung und Schuhe   102.     100  102.   102.   106. 
##  4 Verwendungszwecke des Individual… Wohnung, Wasser, Stro…   99      100  102.   109.   114. 
##  5 Verwendungszwecke des Individual… Möbel, Leuchten, Gerä…   99.9    100  103.   110.   118. 
##  6 Verwendungszwecke des Individual… Gesundheit               98.7    100  100.   102.   105. 
##  7 Verwendungszwecke des Individual… Verkehr                 102.     100  108.   120    124. 
##  8 Verwendungszwecke des Individual… Post und Telekommunik…  102.     100   99.4   99.4   99.8
##  9 Verwendungszwecke des Individual… Freizeit, Unterhaltun…  100.     100  103.   108.   114  
## 10 Verwendungszwecke des Individual… Bildungswesen           100.     100  102.   105.   109. 
## # ℹ 431 more rows
## # ℹ abbreviated name: ¹​`2_variable_attribute_label`
```

# Beispiel 3

``Tabelle: 21611-0002``

**Kinos, Leinwände, Sitzplätze der Kinos, Filmbesuche, Durchschnittlicher Kino-Eintrittspreis, Einnahmen, Filmabgabe: Deutschland, Jahre**


### Das bisherige ffcsv-Format
- Statistik- und sprachspezifische (de/en) Spaltenüberschriften, die sich aus Code, Label und Einheit zusammensetzen


```r
tabelle21611_0002alt <- utils::read.csv("21611-0002_de_flat.csv", 
                                        sep = ";", 
                                        dec = ",", 
                                        na.strings = c("-", "x", ".", "/"),
                                        check.names = FALSE)

tabelle21611_0002alt %>%
  dplyr::select("Zeit", 
                "FILM11__Kinos__Anzahl",
                "FILM08__Sitzplaetze_der_Kinos__EUR",
                "FILM05__Filmbesuche__Mill._EUR",
                "FILM10__Filmbesuche_je_Einwohner__Mill._EUR") %>% 
  tibble::as_tibble() %>% 
  utils::tail(1)
```

```
## # A tibble: 1 × 5
##    Zeit FILM11__Kinos__Anzahl FILM08__Sitzplaetze_der_Kinos__EUR FILM05__Filmbesuche__Mill._…¹
##   <int>                 <int>                              <dbl>                         <dbl>
## 1  2022                  1730                               9.26                           722
## # ℹ abbreviated name: ¹​FILM05__Filmbesuche__Mill._EUR
## # ℹ 1 more variable: FILM10__Filmbesuche_je_Einwohner__Mill._EUR <dbl>
```

### Das neue ffcsv-Format

- Getrennte Spalten für Code, Label und Einheit
- Programme statistikübergreifend nutzbar


```r
# Speichere den entpackten Inhalt in einen temporären Ordner
# Kann auch ein dauerhafter Ordner z.B. auf dem Laufwerk sein
extract_dir <- base::tempdir() 

utils::unzip("21611-0002_de_flat.zip",
             exdir = extract_dir)

# Extrahiere die Datei mit Endung _flat.csv aus dem entpackten Inhalt
extracted_file <- base::list.files(extract_dir,
                                   full.names = TRUE,
                                   pattern = "_flat.csv")

tabelle21611_0002neu <- readr::read_delim(file = extracted_file,
                                          delim = ";",
                                          show_col_types = FALSE,
                                          locale = readr::locale(decimal_mark = ",",
                                                                 grouping_mark = "."),
                                          name_repair = "minimal",
                                          col_types = "c",
                                          na = c("-", "x", ".", "/")) 

# Entferne die temporäre csv-Datei aus dem temporären Ordner
base::invisible(base::file.remove(extracted_file))

tabelle21611_0002neu %>%
  dplyr::select("time", 
                "value",
                "value_unit",
                "value_variable_code",
                "value_variable_label",
                "value_q") %>% 
  dplyr::arrange(time, value_variable_code) %>% 
  tibble::as_tibble() %>% 
  utils::tail(9)
```

```
## # A tibble: 9 × 6
##    time     value value_unit value_variable_code value_variable_label                  value_q
##   <dbl>     <dbl> <chr>      <chr>               <chr>                                 <chr>  
## 1  2022   4911    Anzahl     FILM02              Leinwände                             e      
## 2  2022 777844    Anzahl     FILM03              Sitzplätze der Kinos                  e      
## 3  2022     78    Mill.      FILM04              Filmbesuche                           e      
## 4  2022    722    Mill. EUR  FILM05              Bruttoeinnahmen aus dem Filmbesuch    e      
## 5  2022      0.9  Anzahl     FILM07              Filmbesuche je Einwohner              e      
## 6  2022      9.26 EUR        FILM08              Durchschnittlicher Kino-Eintrittspre… e      
## 7  2022      7    Mill. EUR  FILM09              Filmabgabe der Kinos an die FFA       e      
## 8  2022    715    Mill. EUR  FILM10              Nettoeinnahmen aus dem Filmbesuch (e… e      
## 9  2022   1730    Anzahl     FILM11              Kinos                                 e
```


```r
tabelle21611_0002neu %>% 
  dplyr::filter(time > 2016) %>% 
  tidyr::pivot_wider(id_cols = c("value_variable_label", "value_unit"), 
                     names_from = "time", 
                     values_from = "value") %>% 
  tibble::as_tibble() 
```

```
## # A tibble: 9 × 8
##   value_variable_label                 value_unit  `2022`  `2021`  `2020` `2019` `2018` `2017`
##   <chr>                                <chr>        <dbl>   <dbl>   <dbl>  <dbl>  <dbl>  <dbl>
## 1 Leinwände                            Anzahl     4.91e+3 4.93e+3 4.93e+3 4.96e3 4.85e3 4.80e3
## 2 Bruttoeinnahmen aus dem Filmbesuch   Mill. EUR  7.22e+2 3.73e+2 3.18e+2 1.02e3 8.99e2 1.06e3
## 3 Filmbesuche                          Mill.      7.8 e+1 4.21e+1 3.81e+1 1.19e2 1.05e2 1.22e2
## 4 Durchschnittlicher Kino-Eintrittspr… EUR        9.26e+0 8.87e+0 8.35e+0 8.63e0 8.54e0 8.63e0
## 5 Kinos                                Anzahl     1.73e+3 1.72e+3 1.73e+3 1.73e3 1.67e3 1.67e3
## 6 Filmbesuche je Einwohner             Anzahl     9   e-1 5   e-1 5   e-1 1.4 e0 1.3 e0 1.5 e0
## 7 Sitzplätze der Kinos                 Anzahl     7.78e+5 7.90e+5 7.94e+5 7.98e5 7.96e5 7.89e5
## 8 Filmabgabe der Kinos an die FFA      Mill. EUR  7   e+0 4   e+0 7.6 e+0 2.06e1 2   e1 2.35e1
## 9 Nettoeinnahmen aus dem Filmbesuch (… Mill. EUR  7.15e+2 3.69e+2 3.10e+2 1.00e3 8.79e2 1.03e3
```

