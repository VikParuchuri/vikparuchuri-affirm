Title: Analyzing US Government Contract Awards in R
Date: 2012-01-23 18:47
Slug: analyzing-us-government-contract-awards
Modified: 2014-01-07 15:33
Status: published
Category: 
Tags: data analysis, government, spending, R


<div class='post'>
As I was exploring open data sources, I came across <a href="http://usaspending.gov/">USA spending</a>. This site contains information on US government contract awards and other disbursements, such as grants and loans. In this post, we will look at data on contracts awarded in the state of Maryland in the fiscal year 2011, which is available by selecting "Maryland" as the state where the contract was received and awarded <a href="http://usaspending.gov/data?carryfilters=on">here</a>. I will use Maryland as a proxy for the nation, as the data set for the whole nation will be a bit more unwieldy to analyze, and the USA spending site appears to need a significant amount of time to generate the data file for it. We may take a look at the data for the whole nation later on. <br /><br />First, we download the data file(leave all the options on the usa spending site the same, except select Maryland as the state where the contracts were received and performed when you download the file if you want to follow along), and read it into R: <br /><pre>spend<-read.csv(file="marylandspendingbasic.csv",<br />head=TRUE,sep=",",stringsAsFactors=FALSE,strip.white=TRUE)<br /></pre>By using the names() function, we can see that the data file has a lot of interesting columns. One column is called extentcompeted, and has values indicating how much competition was involved in the bidding process for the contracts. Understandably, having an open bidding process is in the public interest as it can reduce taxpayer costs. We will start by looking at competition in the bidding process for contracts: <br /><pre>library(ggplot2)<br />competition<-tapply(spend$ExtentCompeted,spend$ExtentCompeted,<br />function(x) length(x)/length(spend$ExtentCompeted))<br /><br />qplot(names(competition),weight=competition*100,xlab="Competition Type",<br />ylab="Percent of Contracts",fill=names(competition)) <br />+ opts(axis.text.x = theme_blank()) <br />+ scale_fill_discrete("Competition Type")<br /></pre>This lets us see what percentage of the data falls into each competition category, and then graphs it using the excellent ggplot2 package.<br /><div class="separator" style="clear: both; text-align: center;"><a href="http://3.bp.blogspot.com/-juBIDA_U5jo/Tx4UTUZZmVI/AAAAAAAAACs/JYhrg-HsobA/s1600/competition.png" imageanchor="1" style="margin-left: 1em; margin-right: 1em;"><img border="0" src="http://3.bp.blogspot.com/-juBIDA_U5jo/Tx4UTUZZmVI/AAAAAAAAACs/JYhrg-HsobA/s1600/competition.png" /></a></div>This shows us that about 58% of contracts went through a full competitive bidding process, whereas only approximately 15% of the contracts did not undergo any kind of competition process.<br /><br />Now, lets see how many dollars of spending fall into each competition category: <br /><pre>comp_dollars<-tapply(spend$DollarsObligated,spend$ExtentCompeted,<br />function(x) sum(x)/sum(spend$DollarsObligated,na.rm=TRUE))<br />qplot(names(comp_dollars),weight=comp_dollars*100,xlab="Competition Type",<br />ylab="Percentage of Dollars Obligated",fill=names(comp_dollars)) <br />+ opts(axis.text.x = theme_blank()) <br />+ scale_fill_discrete("Competition Type")<br /></pre><div class="separator" style="clear: both; text-align: center;"><a href="http://2.bp.blogspot.com/-qS7ggV--ErM/Tx4OGl6XebI/AAAAAAAAACY/Pd4_DX-GWRM/s1600/comp_dollars.png" imageanchor="1" style="margin-left: 1em; margin-right: 1em;"><img border="0" src="http://2.bp.blogspot.com/-qS7ggV--ErM/Tx4OGl6XebI/AAAAAAAAACY/Pd4_DX-GWRM/s1600/comp_dollars.png" /></a></div>This plot tells a very different story, showing that about 31% of all dollars that were spent by the government went to contracts that did not involve competition. Further, only 44.5% of all dollars that were obligated went to contracts that were bid on under a fair and open competitive bidding process.<br /><br />This indicates that large contracts tend to receive less bidding than small contracts. This is strange, as large contracts are the ones that are more likely to have reduced costs as a result of competitive bidding, simply because saving 5% of a larger sum is preferable to saving 5% of a smaller sum. Let's take a look at where these large no-bid contracts are going.<br /><br /><pre>companies<-sort(tapply(spend$DollarsObligated,spend$RecipientName,sum))<br />top_companies<-tail(companies/sum(spend$DollarsObligated),10)*100<br />sum(spend$DollarsObligated/1e9)<br />qplot(names(top_companies),weight=top_companies,xlab="Company",<br />ylab="Percentage of Total Dollars Obligated",fill=names(top_companies)) <br />+ opts(axis.text.x = theme_blank()) <br />+ scale_fill_discrete("Company Name") <br />+ opts(legend.text = theme_text(size = 8))<br /></pre><div class="separator" style="clear: both; text-align: center;"><a href="http://2.bp.blogspot.com/-LOlrhnotsDA/Tx4VdLZhxOI/AAAAAAAAAC4/W8eG1lTJBLs/s1600/companies.png" imageanchor="1" style="margin-left: 1em; margin-right: 1em;"><img border="0" src="http://2.bp.blogspot.com/-LOlrhnotsDA/Tx4VdLZhxOI/AAAAAAAAAC4/W8eG1lTJBLs/s1600/companies.png" /></a></div> First, we note that 15.7 billion dollars in federal contracts were obligated to vendors in the state of Maryland in the fiscal year 2011. Next, we note that about 11.5% of the total federal money allocated in contracts went to Lockheed Martin! In fact, the top 10 companies in terms of dollar value of contracts received were given 41% of all the contracted dollars, which amounts to about 6.5 billion dollars. The top 100 contractors received 72% of all contracted dollars. Now, it is becoming clearer where the large no-bid contracts are going. We will look more in depth at the contracts that did not involve competitive bidding to zero in on the issue: <pre><br />nc_spend<-spend[spend$ExtentCompeted=="Not Competed under SAP" <br />| spend$ExtentCompeted=="Not Competed" <br />| spend$ExtentCompeted=="Not Available for Competition" <br />| spend$ExtentCompeted=="Full and Open Competition after exclusion of sources",]<br />nc_companies<-sort(tapply(nc_spend$DollarsObligated,nc_spend$RecipientName,sum))<br />nc_top_companies<-tail(nc_companies/sum(nc_spend$DollarsObligated),10)*100<br />sum(nc_spend$DollarsObligated)/1e9<br />sum(tail(nc_companies,10))/sum(nc_spend$DollarsObligated)<br /></pre> This tells us that out of the 7.6 billion dollars in government contracts that were disbursed with limited or no competition, 55% of these went to the top 10 contractors who received contracts with limited or no competition. This shows that large contractors tend to receive a much greater share of no compete contracts than other contractors, as the top 10 contractors only received 41% of all federal dollars, yet received 55% of contracts with limited competition. <pre><br />mean(nc_spend$DollarsObligated)<br />mean(spend[spend$ExtentCompeted=="Full and Open Competition",]$DollarsObligated)<br /></pre> This shows that the average contract size with no or limited competition was $253507.1, whereas the average contract size with full and open competition was $113936.3, meaning that, on average, contracts twice as large are given out with no competition.<br/><br/> I will leave this analysis here for now, but please feel free to continue on if you wish. This is a very interesting data set, and I may come back to it if I have time down the line.</div>