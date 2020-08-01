# Data-crawling code : R, rvest package

## 문화체육관광부 추천도서 list (2010~현재 : 크롤링)

library("rvest")
library("dplyr")


spot<-proc.time()
url_front <- 'https://www.mcst.go.kr/kor/s_culture/book/bookList.jsp?pSeq=&pDetailSeq=&pMenuCd=0531000000&pCurrentPage=1&'
url_back <- '&pCategory=00&pSearchType=TITLE&pSearchWord='

## full url stacks
url_stack<-c()
times<-c()
for (year in 2010:2020){
  for (month in 1:12){
    if (month<=9){
      month<-paste0(0, month)
    }
   add<-paste0(url_front, paste0('pRegYear=', year, "&pRegMonth=", month), url_back)
   url_stack<-c(url_stack, add)
   time<-paste0(year, month)
   times<-c(times, time)
  }
}

## html : copy selector 활용하여 node 찾고 내용 scrap
current_time = 127
Book_List<-c()
for (i in (1:current_time)){
  cat<-read_html(url_stack[i]) %>% html_nodes(css='p.cate') %>% html_text()
  book<-read_html(url_stack[i]) %>% html_nodes(css='p.title') %>% html_text()
  
  ## 저자, 출판사 link의 경우 jpg 이미지가 같이 들어있음.
  
  auth<-read_html(url_stack[i]) %>% html_nodes('.contentWrap') %>% html_nodes(css='li:nth-child(1)') 
  auth<-auth[-grep("\t\t", auth)] %>% html_text()
  for (j in (1:length(auth))){
    tmp<-strsplit(auth[j], ":")[[1]][2]
    auth[j]<-trimws(tmp)
  }
  
  
  cpy<-read_html(url_stack[i]) %>% html_nodes('.contentWrap') %>% html_nodes(css='li:nth-child(2)')
  cpy<-cpy[-grep("\t\t", cpy)] %>% html_text()
  for (j in (1:length(cpy))){
    tmp<-strsplit(cpy[j], ":")[[1]][2]
    cpy[j]<-trimws(tmp)
  }
  ref_num<-length(book)
  tmp_time<-rep(times[i], ref_num)
  
  tmp<-cbind(cat, book, auth, cpy, tmp_time)
  Book_List<-rbind(Book_List, tmp)
}

Book_List<-data.frame(Book_List)
names(Book_List)<-c("분야", "제목", "저자", "출판사", "추천연월")

save(Book_List, file = "ref.R")



## Runtime
proc.time()-spot
