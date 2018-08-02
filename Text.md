R Training 180802 Ryosuke TAJIMA 
==============  
# 1.Data control  
* This is boring but important.  
  
## 1-1. Read dataset  
### Setup directory  
- Check the directory before reading data: the default directry: home folder on mac
  
```R  
getwd()  
```  
- You can change the using directory.  File > Change Directory ...  
  
###  Read test dataset  
```R  
d1<-read.table("Test1.txt", header=T) # set 1st row as header  
d1  
head(d1) # this dataset is small. For big dataset, you had better use this for checking dataset  
```  
  
## 1-2. Check read data  
### Outline
```R  
nrow(d1) # total row number  
ncol(d1) # total column number  
dim(d1) # row and column numbers  
```  
  
### Check the class of column  
```R  
class(d1$Stage)  
class(d1$Rep)  
class(d1$SDW)  
```  
  
- Class is important for statical analysis.  
- For correlation and regression analyses, All variables need to be "numeric" class.  
- For ANOVA, explanatory variable needs to be "factor" class and response variable needs to be "numeric" class.  
  
## 1-3. Get some values in data  
### Using row and column number  
```R  
A<-d1[1,1] # [row, column]  
A<-d1[10,1]  
A<-d1[1:3,4:7]  
A<-d1[3,]  
A<-d1[,8]  
A<-d1[1:4,]  
A<-d1[,1:4]  
```  
  
### Using the header name
```R  
Stage<-d1$Stage  
Stage<-d1$stage #NULL, S ≠ s in R  
```  
  
## 1-4. Make subset  
- It's so important.
  
```R  
sd5<-subset(d1,d1$Stage=="5week")  
sd8<-subset(d1,d1$Stage=="8week")  
```  
  
# 2. Check outline of your data using plot  
- Visualization is important.
- See [Anscombe's_quartet](https://en.wikipedia.org/wiki/Anscombe%27s_quartet)  
  
## 2-1. Correlation plots  
```R  
plot(d1[,5:8])  
plot(d1[,6], d1[,7]) # Selected  
plot(d1$SDW, d1$RDW) # Selected  
#Selected two plot  
par(mfrow=c(1,2)) # c(row, column)  
plot(d1[,6], d1[,7])  
plot(d1$SDW, d1$RDW)  
```  
  
## 2-1. Boxplot  
```R  
boxplot(d1$SDW)  
boxplot(sd5$SDW, sd8$SDW)  
boxplot(sd5$SDW~sd5$Limitation)  
boxplot(sd5$SDW~sd5$Inoculation)  
```  
  
# 3. Statical analysis  
## 3-1. ANOVA  
### One-way  
```R  
result<-aov(SDW~Limitation, sd5)  
summary(result)  
```  
  
### Two-way  
```R  
result<-aov(SDW~Limitation+Inoculation+Limitation*Inoculation, sd5)  
summary(result)  
result<-aov(SDW~Limitation*Inoculation, sd5) #short version  
summary(result)  
```  
  
### Three-way  
```R  
result<-aov(SDW~Stage*Limitation*Inoculation, d1) #short version  
summary(result)  
```  
  
### For randomized block design
```R  
result<-aov(SDW~Rep+Limitation*Inoculation, sd5)
summary(result)  
```  
  
### For split-plot design  
```R  
result<-aov(SDW~Rep+Limitation+Error(Rep/Limitation)+Inoculation+Limitation*Inoculation, sd5)   
summary(result)  
```  
  
### For strip-plot design  
```R  
result<-aov(SDW~Rep+Limitation+Inoculation+Error(Rep/(Limitation*Inoculation))+Limitation*Inoculation, sd5)   
summary(result)  
```  
  
### Analyze all data using two-way ANOVA  
```R  
# 5week  
summary(aov(SDW~Limitation*Inoculation, sd5))  
summary(aov(RDW~Limitation*Inoculation, sd5))  
summary(aov(SPC~Limitation*Inoculation, sd5))  
summary(aov(RLD~Limitation*Inoculation, sd5))  
  
# 8week  
summary(aov(SDW~Limitation*Inoculation, sd8))  
summary(aov(RDW~Limitation*Inoculation, sd8))  
summary(aov(SPC~Limitation*Inoculation, sd8))  
summary(aov(RLD~Limitation*Inoculation, sd8))  
```  
  
## 3-2. multiple comparison  
### TukeyHSD  
```R  
d2<-read.table("Test2.txt", header=T)
result<-aov(SDW~Treatment, d2)  
summary(result)  
TukeyHSD(result)  
```  
  
### Using Package  
- Using Packages is easier in various analyses.
  
```R  
install.packages("multcomp", dependencies = TRUE) # install packages  
library(multcomp)  
```  
  
### Tukey  
```R  
result<-aov(SDW~Treatment, d2)  
Tukey<-glht(result, linfct=mcp(Treatment="Tukey"))  
summary(Tukey)  
cld(Tukey, level = 0.05, decreasing = TRUE) # attach alphabet  
```  
  
### Dunnett  
```R  
result<-aov(SDW~Treatment, d2)  
Dunnett<-glht(result, linfct=mcp(Treatment="Dunnett"))  
summary(Dunnett)  
```  
  
## 3-3. Correlation  
```R  
cor(sd5$SDW, sd5$SPC)  
allcor<-cor(sd5[,5:8])
allcor
```  
  
## 3-4. Single linear regression  
```R  
result<-lm(SDW~SPC, sd5)  
summary(result)  
```  
- Coefficient of determination (R-squared) is increacing when new explanatory variables are added.  
- Using adjusted R-squared is more adequat.  
  
## 3-5. Multiple linear regression  
### Check multicollinearity  
```R  
allcor  
```  
- This dataset has high correlation coefficient between the variables. 
- In multiple regression, the correlation coefficients between explanatory variables need to be low.
- As they are higher, the analysis is more instability (and the result is unreliable)  
  
### Multiple linear regression  
```R  
result<-lm(SDW~SPC+RLD, sd5)  
summary(result)  
```  
  
## 3-6. Non linear regression  
### Analyze  
  
```R  
d3<-read.table("Test3.txt", header=T)  
x<-d3$Time
y<-d3$Rootlength
plot(x, y)
result<- nls(y ~K/(1+C*exp(-r*x)), start=c(K=150,r=0.01, C=20))
summary(result)
```
### Check the result  
```R  
x1<-c(1:300)
y1<-123.2/(1+43.23*exp(-0.03388*x1))
plot(x,y, xlim = c(0,250),ylim = c(0,130), xlab="", ylab="")
par(new=TRUE)
plot(x1,y1, type='l', xlim = c(0,250),ylim = c(0,130), xlab="", ylab="")
```  
