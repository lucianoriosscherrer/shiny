# shiny
shiny
library(leaflet)
library(shinythemes)
library(Rcmdr)
library(dplyr)
library(shiny)
library(leaflet)
library(leaflet.minicharts)

library(googlesheets)

sheet <-gs_title("teste")
dados<- gs_read_csv(sheet)


# Production columns
prodCols <- names(dados)[6:26]

prodCols1 <- prodCols[c(1,4,7,10,13,16,19)]
prodCols2 <- prodCols[c(2,5,8,11,14,17,20)]
prodCols3 <- prodCols[c(3,6,9,12,15,18,21)]


m <- leaflet(data = dados) %>% addTiles() %>% setView(-46.018361, -20.11054527, zoom = 7) %>% addMeasure(
  position = "bottomleft",
  primaryLengthUnit = "meters",
  primaryAreaUnit = "sqmeters",
  activeColor = "#3D535D",
  completedColor = "#7D4479") %>% addCircleMarkers(~Longitude, ~Latitude , stroke = TRUE, radius = 1,color = "black",
                                                   weight = 5, opacity = 0.95, fill = TRUE, fillColor = "black", fillOpacity = 0.02,popup = ~as.character(COD_PRESTADOR))



colors <- c("#ff0000", "#fcba50", "#238B45")
#####################################################
####

###
ui<-fluidPage(theme = shinytheme("superhero"),
              
              titlePanel("Indicadores para o Anuário Sistema Ocemg"),
              p("Essa aplicação usa dados de indicadores de desempenho  no Ramo Agropecuário",
                "Contém os indicadores anuais das Dimensões Quadro Social, Quadro Funcional, Desempenho Financeiro e Contribuição para a sociedade no periodo de 2013 a  2017."),
              
              sidebarLayout(
                
                sidebarPanel(
                  selectInput("prods1", "Selecione o Indicador", choices = prodCols1, multiple = FALSE),
                  selectInput("prods2", "Selecione o Indicador", choices = prodCols2, multiple = FALSE),
                  selectInput("prods3", "Selecione o Indicador", choices = prodCols3, multiple = FALSE),
                  submitButton("Altera", icon("refresh"))
                ),
                
                mainPanel(
                  leafletOutput("map")
                )
                
              )
)





####################
server<-function(input, output) {
  
  output$map <- renderLeaflet({
    m %>%
      addMinicharts(
        dados$Longitude, dados$Latitude, 
        chartdata = dados[, c(input$prods1, input$prods2,input$prods3)],
        time = dados$Mes,
        colorPalette = colors,
        width = 45, height = 45
      )
  })
  
}
####
shinyApp(ui=ui,server=server)


