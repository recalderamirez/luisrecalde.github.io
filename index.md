---
title: "Pytyvô 2.0"
output: 
  flexdashboard::flex_dashboard:
    theme: cerulean
runtime: shiny
---

```{r setup, include=FALSE}
library(flexdashboard)
library(shiny)
library(tidyverse)
library(plotly)
library(ggplot2)
library(sf)
library(readxl)
library(leaflet)
library(tidyr)
library(maps)
library(sp)
library(rgdal)
library(htmlwidgets)
library(wakefield)
```


```{r benef_map_setup, include = FALSE}
# GENERAL SETUP: MAPA - NUMERO DE BENEFICIARIOS
# POR DEPARTAMENTOS
# Load data
BENEF_DPTO <- read_excel("Distritos_Beneficiarios_collapsed.xlsx")
BENEF_DPTO2 <- read_excel("TODOS_DISTRITOS_GROUPED_DPTOS.xlsx")
# Clean and prepare data
DPTOS_BENEF <- BENEF_DPTO2 %>%
  na.omit() %>%
  group_by(DEPARTAMENTO_COMERCIO, DPTO) %>%
  summarise(USD = sum(USD))
cleaned_benef <- merge(BENEF_DPTO, DPTOS_BENEF, by = "DPTO")
cleaned_benef <- cleaned_benef %>%
    select(DPTO, cantidad, DEPARTAMENTO_COMERCIO)
# Load shapefile (Departamentos de Paraguay)
deptos_benef <- readOGR("Departamentos_Paraguay.shp")
# Cleaning
deptos_benef$DPTO <- as.numeric(deptos_benef$DPTO)
# Merge data with shapefile
merged_dpto_benef <- merge(deptos_benef, cleaned_benef, by = "DPTO")
# POR DISTRITOS
# Load data
BENEF_DIST <- read_excel("TODOS_DISTRITOS_GROUPED.xlsx")
# Clean and prepare data
DIST_BENEF <- BENEF_DIST%>%
  filter(!is.na(CLAVE)) %>%
  group_by(DEPARTAMENTO_COMERCIO, DIST_DESC, CLAVE) %>%
  summarise(MONTO = sum(MONTO),
            USD = sum(USD)) %>%
  select(DEPARTAMENTO_COMERCIO, DIST_DESC, CLAVE)
# Load shapefile (Distritos de Paraguay)
distritos_ben <- readOGR("Distritos_Beneficiarios.shp")
# Further cleaning
distritos_ben$CLAVE <- as.numeric(distritos_ben$CLAVE)
# Merge data with shapefile
merged_dist_benef <- merge(distritos_ben, DIST_BENEF, by = "CLAVE")
```


```{r cons_map_setup, include = FALSE}
# GENERAL SETUP: MAPA - NUMERO DE BENEFICIARIOS
# POR DEPARTAMENTOS
# Load data
CONSUMO_DPTO <- read_excel("TODOS_DISTRITOS_GROUPED_DPTOS.xlsx")
# Clean and prepare data
DPTOS <- CONSUMO_DPTO %>%
  na.omit() %>%
  group_by(DEPARTAMENTO_COMERCIO, DPTO) %>%
  summarise(USD = sum(USD)) %>%
  mutate(TOTAL = round(USD / 1000))
# Load shapefile (Departamentos de Paraguay)
deptos <- readOGR("Departamentos_Paraguay.shp")
# Further cleaning
deptos$DPTO <- as.numeric(deptos$DPTO)
# Merge data with shapefile
merged_dpto <- merge(deptos, DPTOS, by = "DPTO")
# POR DISTRITOS
# Load data
CONSUMO_DIST <- read_excel("TODOS_DISTRITOS_GROUPED.xlsx")
# Clean and prepare data
DIST <- CONSUMO_DIST %>%
  filter(!is.na(CLAVE)) %>%
  group_by(DEPARTAMENTO_COMERCIO, DIST_DESC, CLAVE) %>%
  summarise(MONTO = sum(MONTO),
            USD = sum(USD)) %>%
  mutate(TOTAL = round(USD / 1000))
# Load shapefile (Distritos de Paraguay)
distritos <- readOGR("Distritos_Paraguay.shp")
# Further cleaning
distritos$CLAVE <- as.numeric(distritos$CLAVE)
# Merge data with shapefile
merged_dist <- merge(distritos, DIST, by = "CLAVE")
```

Resumen General {data-icon="fa-chart-pie"}
=============================

Column {data-width=400}
-----------------------------------------------------------------------

### personas beneficiadas

```{r 1}
Beneficiarios <- "763.825"
valueBox(Beneficiarios, icon = "fa-user-friends")
```


### Beneficiarios por sexo

```{r graph}
Sexo_sample <- as.data.frame(rbinom(n=763825, size= 1, prob=0.5))
Sexo_sample <- Sexo_sample %>%
  group_by(`rbinom(n = 763825, size = 1, prob = 0.5)`) %>%
  summarise(cant = n())
labels_sex <- c("Femenino", "Masculino")
plot_ly(Sexo_sample, labels= labels_sex, values = ~cant,
        insidetextfont = list(size = 20),
        showlegend = T) %>%
  add_pie(hole = 0.6) %>%
  layout(xaxis = list(showgrid = F, zeroline = F, showticklabels = F),
         yaxis = list(showgrid = F, zeroline = F, showticklabels = F)) %>% 
  layout(legend= list(orientation="v", font=(list(size=12))),
         xaxis = list(showgrid = F, zeroline = F, showticklabels = F),
         yaxis = list(showgrid = F, zeroline = F, showticklabels = F))
```

### EMPES utilizadas

```{r mapa2}
EMPES <- read_excel("DATOS_PYTYVO_2_1 CONSOLIDADO.xlsx")
EMPES <- EMPES %>%
  group_by(EMPE) %>%
  summarise(n())
colnames(EMPES) <- c("EMPE", "cant")
plot_ly(EMPES, labels= ~EMPE, values = ~cant,
        insidetextfont = list(size = 20),
        showlegend = T) %>%
  add_pie(hole = 0.6) %>%
  layout(xaxis = list(showgrid = F, zeroline = F, showticklabels = F),
         yaxis = list(showgrid = F, zeroline = F, showticklabels = F)) %>% 
  layout(legend= list(orientation="v", font=(list(size=12))),
         xaxis = list(showgrid = F, zeroline = F, showticklabels = F),
         yaxis = list(showgrid = F, zeroline = F, showticklabels = F))
```


Column {data-width=400}
-----------------------------------------------------------------------

### millones de dólares en consumo

```{r 2}
consumo <- "42,4"
valueBox(consumo, icon = "fa-dollar-sign")
```


### Beneficiarios por edad

```{r graph2}
Edad_sample <- as.data.frame(rnorm(770000, mean = 45, sd=7))
colnames(Edad_sample) <- c("edad")
Edad_sample$edad <- round(Edad_sample$edad)
Edad_sample <- Edad_sample %>%
  filter(edad > 17)
Edad_sample$edad <- as.factor(Edad_sample$edad)
plot_ly(Edad_sample,
        x = ~edad,
        histfunc = 'sum',
        type = "histogram") %>%
  layout(xaxis = list(title = "Edad",
                      titlefont = list(size = 12)))
```

### Comercios adheridos

```{r mapa3}
Comercios <- read_excel("DATOS_PYTYVO_2_1 CONSOLIDADO.xlsx")
Comercios$RUBRO[which(Comercios$RUBRO == "DESPENSAS")] <- "Despensas"
Comercios$RUBRO[which(Comercios$RUBRO == "ESTACIONES DE SERVICIO")] <- 
  "Estaciones de Servicio"
Comercios$RUBRO[which(Comercios$RUBRO == "ESTACIONES DE SERVICIOS")] <- 
  "Estaciones de Servicio"
Comercios$RUBRO[which(Comercios$RUBRO == "Estaciones de Servicios")] <- 
  "Estaciones de Servicio"
Comercios$RUBRO[which(Comercios$RUBRO == "FARMACIAS Y PERFUMERIAS")] <- 
  "Farmacias y Perfumerías"
Comercios$RUBRO[which(Comercios$RUBRO == "FARMACIAS")] <- 
  "Farmacias y Perfumerías"
Comercios$RUBRO[which(Comercios$RUBRO == "OTROS")] <- 
  "Otros"
Comercios$RUBRO[which(Comercios$RUBRO == "SUPERMERCADOS")] <- 
  "Supermercados"
Comercios$RUBRO[which(Comercios$RUBRO == "SUPERMERCADOS y DESPENSAS")] <- 
  "Supermercados"
Comercios$RUBRO[which(Comercios$RUBRO == "SUPERMERCADOS Y DESPENSAS")] <- 
  "Supermercados"
Comercios <- Comercios %>%
  group_by(RUBRO) %>%
  summarise(n())
colnames(Comercios) <- c("RUBRO", "cant")
plot_ly(Comercios, x = ~RUBRO, y = ~cant, type = 'bar',
        marker = list(color = 'rgb(158,202,225)',
                      line = list(color = 'rgb(8,48,107)',
                                  width = 1.5))) %>% 
  layout(xaxis = list(title = "Tipo de Comercios",
                      showticklabels = F,
                      titlefont = list(size = 12)),
         yaxis = list(title = "Cantidad",
                      titlefont = list(size = 12)))
```


Column {data-width=200}{.tabset .tabset-fade} 
-----------------------------------------------------------------------


### Cantidad de Beneficiarios

```{r benef_map}
# DEPARTAMENTOS
# intervals for distribution
bins_dpto <- c(1000, 7000, 15000, 30000, 50000, 100000, 200000)
# color palettes
pal_dpto_benef <- colorBin(palette = "YlOrBr",
                domain = merged_dist_benef$cantidad, bins = bins_dpto, reverse = F)
# html code for label popup
labels_dpto <- sprintf(
  "<strong>Departamento:</strong> %s <br><strong>Nro. total de beneficiarios:</strong> %g",
  merged_dpto_benef$DEPARTAMENTO_COMERCIO, merged_dpto_benef$cantidad) %>%
  lapply(htmltools::HTML)
# Leaflet Map
mapa_benef1 <- leaflet(merged_dpto_benef) %>%
  addProviderTiles(provider = "CartoDB.Positron") %>%
  addPolygons(
    fillColor = ~ pal_dpto_benef(cantidad),
    weight = 0.5,
    opacity = 1,
    color = "#3b3e45",
    dashArray = "",
    fillOpacity = 0.9,
    highlight = highlightOptions(
      weight = 2,
      color = "#666",
      dashArray = "",
      fillOpacity = 0.5,
      bringToFront = T),
    label = labels_dpto,
    labelOptions = labelOptions(
      style = list("font-weight" = "normal", padding = "3px 8px"),
      textsize = "12px",
      direction = "auto")) %>%
  addLegend("topright",
            pal = pal_dpto_benef, 
            values = ~ cantidad,
            title = "Nro. de beneficiarios",
            opacity = 0.7) %>%
  addScaleBar("bottomleft", options =
                scaleBarOptions(imperial = T, updateWhenIdle = T))
# DISTRITOS
# intervals for distribution
bins_dist <- c(0, 500, 1000, 10000, 20000, 50000, 120000)
# color palettes
pal_dist <- colorBin(palette = "YlGnBu",
                domain = merged_dist_benef$cantidad, bins = bins_dist, reverse = F)
# html code for label popup
labels_dist <- sprintf(
  "<strong>Distrito:</strong> %s <br><strong>Nro. total de beneficiarios:</strong> %g",
  merged_dist_benef$DIST_DESC.y, merged_dist_benef$cantidad) %>%
  lapply(htmltools::HTML)
# Leaflet Map
mapa_benef2 <- leaflet(merged_dist_benef) %>%
  addProviderTiles(provider = "CartoDB.Positron") %>%
  addPolygons(
    fillColor = ~ pal_dist(cantidad),
    weight = 0.5,
    opacity = 1,
    color = "#3b3e45",
    dashArray = "",
    fillOpacity = 0.9,
    highlight = highlightOptions(
      weight = 2,
      color = "#666",
      dashArray = "",
      fillOpacity = 0.5,
      bringToFront = T),
    label = labels_dist,
    labelOptions = labelOptions(
      style = list("font-weight" = "normal", padding = "3px 8px"),
      textsize = "12px",
      direction = "auto")) %>%
  addLegend("topright",
            pal = pal_dist, 
            values = ~ cantidad,
            title = "Nro. de beneficiarios",
            opacity = 0.7) %>%
  addScaleBar("bottomleft", options =
                scaleBarOptions(imperial = T, updateWhenIdle = T))
# Shiny code
ui <- fluidPage(
      radioButtons(inputId = "mapbenef",
                   label = "Seleccione",
                            c("Por Departamento" = "dept",
                              "Por Distrito" = "dist"),
                   inline = T),
      leafletOutput("mapabenef", height = 650))
server <- function(input, output) {
  output$mapabenef <- renderLeaflet({
    mapbenef <- switch (input$mapbenef,
                        dept = mapa_benef1,
                        dist = mapa_benef2,
                        mapa_benef1)
  })
  }
shinyApp(ui = ui, server = server)
```


### Consumo en Comercios

```{r cons_map}
# DEPARTAMENTOS
# intervals for distribution
bins_dpto <- c(0, 500, 1000, 5000, 8000, 10000, 13000)
# color palettes
pal_dpto <- colorBin(palette = "PuRd",
                domain = merged_dist$TOTAL, bins = bins_dpto, reverse = F)
# html code for label popup
labels_dpto <- sprintf(
  "<strong>Distrito:</strong> %s <br><strong>Consumo total en miles de USD:</strong> %g",
  merged_dpto$DEPARTAMENTO_COMERCIO, merged_dpto$TOTAL) %>%
  lapply(htmltools::HTML)
# Leaflet Map
mapa1 <- leaflet(merged_dpto) %>%
  addProviderTiles(provider = "CartoDB.Positron") %>%
  addPolygons(
    fillColor = ~ pal_dpto(TOTAL),
    weight = 0.5,
    opacity = 1,
    color = "#3b3e45",
    dashArray = "",
    fillOpacity = 0.9,
    highlight = highlightOptions(
      weight = 2,
      color = "#666",
      dashArray = "",
      fillOpacity = 0.5,
      bringToFront = T),
    label = labels_dpto,
    labelOptions = labelOptions(
      style = list("font-weight" = "normal", padding = "3px 8px"),
      textsize = "12px",
      direction = "auto")) %>%
  addLegend("topright",
            pal = pal_dpto, 
            values = ~ TOTAL,
            title = "Consumo en miles de USD",
            opacity = 0.7) %>%
  addScaleBar("bottomleft", options =
                scaleBarOptions(imperial = T, updateWhenIdle = T))
# DISTRITOS
# intervals for distribution
bins_dist <- c(0, 15, 35, 65, 160, 500, 4500)
# color palettes
pal_dist <- colorBin(palette = "Blues",
                domain = merged_dist$TOTAL, bins = bins_dist, reverse = F)
# html code for label popup
labels_dist <- sprintf(
  "<strong>Distrito:</strong> %s <br><strong>Consumo total en miles de USD:</strong> %g",
  merged_dist$DIST_DESC.y, merged_dist$TOTAL) %>%
  lapply(htmltools::HTML)
# Leaflet Map
mapa2 <- leaflet(merged_dist) %>%
  addProviderTiles(provider = "CartoDB.Positron") %>%
  addPolygons(
    fillColor = ~ pal_dist(TOTAL),
    weight = 0.5,
    opacity = 1,
    color = "#3b3e45",
    dashArray = "",
    fillOpacity = 0.9,
    highlight = highlightOptions(
      weight = 2,
      color = "#666",
      dashArray = "",
      fillOpacity = 0.5,
      bringToFront = T),
    label = labels_dist,
    labelOptions = labelOptions(
      style = list("font-weight" = "normal", padding = "3px 8px"),
      textsize = "12px",
      direction = "auto")) %>%
  addLegend("topright",
            pal = pal_dist, 
            values = ~ TOTAL,
            title = "Consumo en miles de USD",
            opacity = 0.7) %>%
  addScaleBar("bottomleft", options =
                scaleBarOptions(imperial = T, updateWhenIdle = T))
# Shiny code
ui <- fluidPage(
      radioButtons(inputId = "map1",
                   label = "Seleccione",
                            c("Por Departamento" = "dept",
                              "Por Distrito" = "dist"),
                   inline = T),
      leafletOutput("mapa1", height = 650))
server <- function(input, output) {
  output$mapa1 <- renderLeaflet({
    map1 <- switch (input$map1,
                    dept = mapa1,
                    dist = mapa2,
                    mapa1)
  })
  }
shinyApp(ui = ui, server = server)
```


Sobre el Programa {data-icon="fa-question"}
=============================

### **[EJEMPLO] Pytyvõ 2.0: Habilitan web y App para que potenciales beneficiarios se inscriban a subsidio**

[ejemplo]
La iniciativa tiene el propósito de extender el modo seguro de vivir a toda la ciudadanía. Foto: Agencia IP
Asunción, Agencia IP. – El Ministerio de Hacienda habilitó el portal y web para que los potenciales beneficiarios del programa de subsidio alimentario Pytyvõ 2.0 puedan inscribirse. Hoy corresponde a los que tienen el número de cédula con  0 y 1.

Los requisitos para acceder al subsidio son: contar con Cédula de Identidad paraguaya, ser mayor de 18 años, no ser funcionario público ni cotizar a la seguridad social y no ser titular de ningún otro programa de asistencia social del Estado como Adultos Mayores y Tekoporã.

Estar registrado en la Subsecretaría de Estado de Tributación (SET) no será un requisito excluyente, sin embargo, se priorizará a aquellos que hayan declarado hasta un salario mínimo como ingreso.

Además de ese punto, los formularios de inscripción incorporan un apartado sobre el entorno de convivencia del potencial beneficiario. Allí el trabajador deberá declarar los datos de las personas con las que convive y con quienes comparte los gastos del hogar, ya que el subsidio podrá ser otorgado hasta a dos personas del mismo hogar, siempre y cuando cumplan con las condiciones.

Cabe destacar, finalmente, que el Gobierno Nacional puso especial celeridad para reglamentar la Ley Nº 6587/2020 y así poder poner en marcha lo antes posible el programa de subsidio Pytyvõ 2.0, una asistencia monetaria de 500.000 guaraníes destinada a la compra de alimentos y productos de higiene destinada a los trabajadores informales y cuentapropistas de ciudades fronterizas y de todo el país, afectados por el impacto de la pandemia del covid-19.

Para este lunes 10 de agosto podrán inscribirse todos aquellos cuya terminación 0 y 1; el martes 11, culminación 2 y 3; miércoles 12, a los correspondientes 4 y 5.

En tanto para el jueves 13, los que terminen en 6 y 7; el viernes 14, los que sus documentos concluyan con 8 y 9; y el sábado 15 será para las terminaciones pendientes, que son las personas que no se hayan registrado o cuenten con cédula alfanumérica.

El programa Pytyvõ 2.0 cuenta con un fondo de 125 millones de dólares y constará de dos pagos de 500.000 guaraníes, y, conforme a disponibilidad presupuestaria y financiera, eventualmente un tercer y cuarto pago de igual monto, con lo cual se pretende beneficiar a más de 700.000 trabajadores. El mismo está destinado a trabajadores informales, cuentapropistas y trabajadores dependientes de micro, pequeñas y medianas empresas afectados en sus ingresos por la pandemia del covid-19.

Para inscribirse por medio de la web del Ministerio de Hacienda puedes seguir el enlace.

Para descargar la aplicación al celular puedes seguir el siguiente enlace.
