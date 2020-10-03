# Web Scraping
## Recolectar data de laptop de la tienda Wong
1. codigo.R
	```r
	library(rvest)
	url<-"https://www.wong.pe/busca/?ft=laptop"
	pagina <- read_html(url)
	selector <- "div.product-item__bottom > div.product-item__info > a"
	nodo <- html_nodes(pagina,selector)
	text <- html_text(nodo)
	links <- html_attr(nodo,"href")
	links

	getArticulo<-function(url){
	#url<-"https://www.wong.pe/fiddler-cooler-para-laptop-15-785296/p"
	pagina_web<-read_html(url)
	nombre<-"#product-nonfood-page > div.wrapper.product-wrapper > div > div.product-content > div.inner.product > div.product-info > div.info-wrapper > div.name > div"
	nombre_nodo<-html_node(pagina_web, nombre)
	nombre_texto<-html_text(nombre_nodo)

	marca<-"#product-nonfood-page > div.wrapper.product-wrapper > div > div.product-content > div.inner.product > div.product-info > div.info-wrapper > div.aditional-info > span.brand > div > a"
	marca_nodo<-html_node(pagina_web,marca)
	marca_texto<-html_text(marca_nodo)

	valoraciones<-"#product-nonfood-page > div.wrapper.product-wrapper > div > div.product-content > div.inner.product > div.product-info > div.info-wrapper > div.reviews__rating.reviews__rating--product-wrapper.desktop > div.reviews__rating-amount > div"
	valoraciones_nodo<-html_node(pagina_web,valoraciones)
	valoraciones_texto<-html_text(valoraciones_nodo)

	codigo<-"#product-nonfood-page > div.wrapper.product-wrapper > div > div.product-content > div.inner.product > div.product-info > div.info-wrapper > div.aditional-info > span.code > div"
	codigo_nodo<-html_node(pagina_web,codigo)
	codigo_texto<-html_text(codigo_nodo)

	articulo<-c(nombre_texto,marca_texto,valoraciones_texto,codigo_texto)
	articulo
	}
	datos<-sapply(links,getArticulo)
	b<-t(datos)
	colnames(b)<-c("Nombre de producto","Marca","Valoraciones","Código de producto")
	b
	write.csv(b,"laptop.csv")
	```
2. Output
	![](.img/datalaptop.png)
## Scrolling
1. codigo.R
	```r
	library(RSelenium)
	library(rvest)
	#start RSelenium
	driver<-rsDriver(port=4569L,browser="chrome",chromever="85.0.4183.83")
	remDr <- driver[["client"]]
	#navigate to your page
	remDr$navigate("https://www.wong.pe/abarrotes")
	#scroll down 5 times, waiting for the page to load at each time
	for(i in 1:60){
	remDr$executeScript(paste("scroll(0,",i*10000,");"))
	Sys.sleep(3)    
	}
	#get the page html
	page_source<-remDr$getPageSource()
	#parse it
	a<-read_html(page_source[[1]]) %>% html_nodes(".product-item__info") %>% html_text()
	b<-gsub('[\r\n\t]','',a)
	write.csv(b,"productos.csv")
	```
2. Output  
	![](.img/output2.png)  
	![](.img/output3.png)  
## Paginación
1. codigo.R
	```
	library(rvest)
	paginas<-paste0("https://www.amazon.es/s?k=caballo&page=",c(1:2),"&__mk_es_ES=%C3%85M%C3%85%C5%BD%C3%95%C3%91&qid=1601617471&ref=sr_pg_",c(1:2))
	linksPagina<-function(url){
	#url<-"https://www.amazon.es/s?k=caballo&page=3&__mk_es_ES=%C3%85M%C3%85%C5%BD%C3%95%C3%91&qid=1601617471&ref=sr_pg_3"
	pagina<-read_html(url)
	selector<-"div > div > div:nth-child(3) > h2 > a"
	nodo<-html_nodes(pagina,selector)
	links<-html_attr(nodo,"href")
	links<-paste0("https://amazon.es/",links)
	}

	datosProducto<-function(url){
	#library(rvest)
	#url<-"https://www.amazon.es/Happy-People-58980-importado-Alemania/dp/B003YGDCBQ/ref=sr_1_102?__mk_es_ES=%C3%85M%C3%85%C5%BD%C3%95%C3%91&dchild=1&keywords=caballo&qid=1601617471&sr=8-102"
	pagina<-read_html(url)
	#nombre
	nombre<-"#productTitle"
	nombre_nodo<-html_node(pagina,nombre)
	nombre_texto<-html_text(nombre_nodo)
	#precio
	precio<-"#priceblock_ourprice"
	precio_nodo<-html_node(pagina,precio)
	precio_texto<-html_text(precio_nodo)
	#valoracion
	valoracion<-"#acrCustomerReviewText"
	valoracion_nodo<-html_node(pagina,valoracion)
	valoracion_texto<-html_text(valoracion_nodo)
	#estrellas
	estrellas<-"#acrPopover > span.a-declarative > a > i.a-icon.a-icon-star.a-star-4"
	estrellas_nodo<-html_node(pagina,estrellas)
	estrellas_texto<-html_text(estrellas_nodo)
	producto<-c(nombre_texto,precio_texto,valoracion_texto,estrellas_texto)
	producto
	}

	links<-sapply(paginas,linksPagina)
	a<-as.vector(unlist(links))
	datos<-sapply(a,datosProducto)
	names(datos)<-NULL
	b<-t(datos)
	colnames(b)<-c("Nombre del producto","Precio","valoraciones","estrellas")
	rownames(b)<-NULL
	c<-gsub('[\n]','',b[,1])
	d<-cbind(c,b[,2:4])
	write.csv(d,"paginacion.csv")
	```
2. Output
	![](.img/resultado.png)
