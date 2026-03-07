---
title: CLASS PROJECT高级程序设计C++
date: 2026-01-01 08:07:00
tags:
  - "C++"
  - "编程"
  - "高级程序设计"
categories:
  - "开发工具"
---

﻿# 高级程序设计C++作业
## 按照如下要求建立程序，并演示程序运行结果：    

 1. 用名称、人口、海拔高度、天气、年份等数据成员建立一个名为City的类。建立一个产生City对象的类。将产生的City对象（数量大于1000个）填充至一个容器，容器的类型自选。对于City对象的具体属性值通过创建发生器来生成。生成规则如下：年份为2009年；名称由4-8个英文小写字符随机构成；人口在范围[300000,10000000)内随机选取；海拔高度在范围[0,5000)米内随机选取；上述三值均不可重复；天气在枚举常量表中{Rainy,Snowy,Cloudy,Sunny}随机选取（1年天气取12个值，即每月一个值）。容器填充完毕后，将其内容写入一个名为City.txt的文件。
 2. 从2009年至2019年间，各城市人口按照n%的速度进行变化。以题目1中生成数据作为2009年数据计算各城市从2009年到2019年各年的人口数，其中n的值在-10到+10间随机选取。计算完毕后将数据重新写回文件City.txt。注意：可按照年份存储10个文件，依次存储10年的数据（文件名依次为City2009,City2010…）。也可将所有数据存储在一个文件中；每年的天气数据按照题目1的规则同样生成。
 3. 设计算法，对2009年至2019年间的各城市按照其人口数进行查找，找出这10年里人口最多、最少和人口处于中位数的各个城市。结果写入文件Population.txt，格式为“年份，最多人口城市名称，最少人口城市名称，中位数人口城市名称” 。
 4. 设计算法，查找2009年到2019年的10年间，每年拥有最好天气数量的城市（即Sunny最多），结果写入文件Weather.txt，格式为“年份，城市名称，Sunny数量”。

```cpp
#include<iostream>
#include<vector>
#include<string>
#include<time.h>
#include<algorithm>
using namespace std;
enum Enum_weather       //天气在枚举常量中表示
{
	Rainy=0,Snowy,Cloudy,Sunny
};
class City      //创建城市类
{
public:
	int year;

	string name;
	int persons;
	int elevation;
	char *weather[12];
	City()
	{
	}
	~City(){}
};
char *rand_str(char *str)     //随机生成字符串函数
{
	int i,n;
	n=rand()%5;
	for(i=0;i<n+4;++i)
	{
		str[i]='a'+rand()%26;
	}	
	str[++i]='\\0';
	return str;
}
static inline char* weather_str(enum Enum_weather w)      //枚举类型转化为char*
{  
	char *strings[] = {"Rainy", "Snowy", "Cloudy", "Sunny",};  
	return strings[w];  
}
int main()
{
    int i,j; 
	srand((unsigned)time(NULL)); 
	vector<City>ve(1023);
	for ( i=0;i<1023;i++)  //将随机获取每个城市的数据写入vector中
	{
		ve[i].year=2009;
		ve[i].persons=rand()%(10000000 - 300000 + 1) + 300000;  //在某范围内随机获取数值
		ve[i].elevation=rand()%(5000-0+1)+0;	
		char name1[10]={0};
		rand_str(name1);   //调用rand_str（）函数随机生成城市名称字符串
		ve[i].name=name1;
		for (int j=0;j<12;j++)          
		{
			int n=rand()%4;
			Enum_weather w1=(enum Enum_weather)(n);
			char* nc=weather_str(w1);   //调用weather_str（）函数将枚举类型值转化为char*
			ve[i].weather[j]=nc;
		}
	}
	//将2009年各个城市的数据信息写入City.txt文本文件中
	FILE *fp;
	fp=fopen("D://City.txt","w+");
	fprintf(fp,"年份   城市名称   城市人口   海拔高度   1月天气   2月天气    3月天气    4月天气    5月天气    6月天气    7月天气    8月天气    9月天气    10月天气   11月天气   12月天气\n");
	for ( i=0;i<1023;i++)
	{
		char name2[10]={0};
		for (int k=0;k<ve[i].name.size();k++)    //string类型转char数组
		{
			name2[k]=ve[i].name[k];
		}
		fprintf(fp,"%d%10s%10d%10d",ve[i].year,name2,ve[i].persons,ve[i].elevation);
		for (int j=0;j<12;j++)
		{
			fprintf(fp,"%11s",ve[i].weather[j]);
		}
		fprintf(fp,"\n");
	}
	fclose(fp);
	//从2009年至2019年间各城市数据写入vector中
	vector<City>ve_city1(1023);vector<City>ve_city2(1023);vector<City>ve_city3(1023);vector<City>ve_city4(1023);vector<City>ve_city5(1023);  
	vector<City>ve_city6(1023);vector<City>ve_city7(1023);vector<City>ve_city8(1023);vector<City>ve_city9(1023);vector<City>ve_city10(1023);
	//2009年各城市数据获取，并将数据写入City2009.txt文件中 
	for ( i=0;i<1023;i++)
	{
		ve_city1[i].year=2009;
		ve_city1[i].persons=(1+(rand()%20-10)*0.01)*ve[i].persons; //2009年各城市人口按照n%的速度进行变化，其中n的值在-10到+10间随机选取
		ve_city1[i].elevation=ve[i].elevation;	
		ve_city1[i].name=ve[i].name;
		for (int j=0;j<12;j++)          
		{
			int n=rand()%4;
			Enum_weather w1=(enum Enum_weather)(n);
			char* nc=weather_str(w1); 
			ve_city1[i].weather[j]=nc;
		}
	}
	FILE *fp1;
	fp1=fopen("D://City2009.txt","w+");
	fprintf(fp1,"年份   城市名称   城市人口   海拔高度   1月天气   2月天气    3月天气    4月天气    5月天气    6月天气    7月天气    8月天气    9月天气    10月天气   11月天气   12月天气\n");
	for ( i=0;i<1023;i++)
	{
		char name2[10]={0};
		for (int k=0;k<ve_city1[i].name.size();k++)    //string类型转char数组
		{
			name2[k]=ve_city1[i].name[k];
		}
		fprintf(fp1,"%d%10s%10d%10d",ve_city1[i].year,name2,ve_city1[i].persons,ve_city1[i].elevation);
		for (int j=0;j<12;j++)
		{
			fprintf(fp1,"%11s",ve_city1[i].weather[j]);
		}
		fprintf(fp1,"\n");
	}
	fclose(fp1);
	//2010年各城市数据获取，并将数据写入City2010.txt文件中
	for ( i=0;i<1023;i++)
	{
		ve_city2[i].year=2010;
		ve_city2[i].persons=(1+(rand()%20-10)*0.01)*ve[i].persons; //2010年各城市人口按照n%的速度进行变化，其中n的值在-10到+10间随机选取
		ve_city2[i].elevation=ve[i].elevation;	 
		ve_city2[i].name=ve[i].name;
		for (int j=0;j<12;j++)          
		{
			int n=rand()%4;
			Enum_weather w1=(enum Enum_weather)(n);
			char* nc=weather_str(w1); 
			ve_city2[i].weather[j]=nc;
		}
	}
	FILE *fp2;
	fp2=fopen("D://City2010.txt","w+");
	fprintf(fp2,"年份   城市名称   城市人口   海拔高度   1月天气   2月天气    3月天气    4月天气    5月天气    6月天气    7月天气    8月天气    9月天气    10月天气   11月天气   12月天气\n");
	for ( i=0;i<1023;i++)
	{
		char name2[10]={0};
		for (int k=0;k<ve_city2[i].name.size();k++)    //string类型转char数组
		{
			name2[k]=ve_city2[i].name[k];
		}
		fprintf(fp2,"%d%10s%10d%10d",ve_city2[i].year,name2,ve_city2[i].persons,ve_city2[i].elevation);
		for (int j=0;j<12;j++)
		{
			fprintf(fp2,"%11s",ve_city2[i].weather[j]);
		}
		fprintf(fp2,"\n");
	}
	fclose(fp2);
	//2011年各城市数据获取，并将数据写入City2011.txt文件中
	for ( i=0;i<1023;i++)
	{
		ve_city3[i].year=2011;
		ve_city3[i].persons=(1+(rand()%20-10)*0.01)*ve[i].persons; //2011年各城市人口按照n%的速度进行变化，其中n的值在-10到+10间随机选取
		ve_city3[i].elevation=ve[i].elevation;	
		ve_city3[i].name=ve[i].name;
		for (int j=0;j<12;j++)          
		{
			int n=rand()%4;
			Enum_weather w1=(enum Enum_weather)(n);
			char* nc=weather_str(w1); 
			ve_city3[i].weather[j]=nc;
		}
	}
	FILE *fp3;
	fp3=fopen("D://City2011.txt","w+");
	fprintf(fp3,"年份   城市名称   城市人口   海拔高度   1月天气   2月天气    3月天气    4月天气    5月天气    6月天气    7月天气    8月天气    9月天气    10月天气   11月天气   12月天气\n");
	for ( i=0;i<1023;i++)
	{
		char name2[10]={0};
		for (int k=0;k<ve_city3[i].name.size();k++)    //string类型转char数组
		{
			name2[k]=ve_city3[i].name[k];
		}
		fprintf(fp3,"%d%10s%10d%10d",ve_city3[i].year,name2,ve_city3[i].persons,ve_city3[i].elevation);
		for (int j=0;j<12;j++)
		{
			fprintf(fp3,"%11s",ve_city3[i].weather[j]);
		}
		fprintf(fp3,"\n");
	}
	fclose(fp3);
	//2012年各城市数据获取，并将数据写入City2012.txt文件中
	for ( i=0;i<1023;i++)
	{
		ve_city4[i].year=2012;
		ve_city4[i].persons=(1+(rand()%20-10)*0.01)*ve[i].persons; //2012年各城市人口按照n%的速度进行变化，其中n的值在-10到+10间随机选取
		ve_city4[i].elevation=ve[i].elevation;	
		ve_city4[i].name=ve[i].name;
		for (int j=0;j<12;j++)          
		{
			int n=rand()%4;
			Enum_weather w1=(enum Enum_weather)(n);
			char* nc=weather_str(w1); 
			ve_city4[i].weather[j]=nc;
		}
	}
	FILE *fp4;
	fp4=fopen("D://City2012.txt","w+");
	fprintf(fp4,"年份   城市名称   城市人口   海拔高度   1月天气   2月天气    3月天气    4月天气    5月天气    6月天气    7月天气    8月天气    9月天气    10月天气   11月天气   12月天气\n");
	for ( i=0;i<1023;i++)
	{
		char name2[10]={0};
		for (int k=0;k<ve_city4[i].name.size();k++)    //string类型转char数组
		{
			name2[k]=ve_city4[i].name[k];
		}
		fprintf(fp4,"%d%10s%10d%10d",ve_city4[i].year,name2,ve_city4[i].persons,ve_city4[i].elevation);
		for (int j=0;j<12;j++)
		{
			fprintf(fp4,"%11s",ve_city4[i].weather[j]);
		}
		fprintf(fp4,"\n");
	}
	fclose(fp4);
	//2013年各城市数据获取，并将数据写入City2013.txt文件中
	for ( i=0;i<1023;i++)
	{
		ve_city5[i].year=2013;
		ve_city5[i].persons=(1+(rand()%20-10)*0.01)*ve[i].persons; //2013年各城市人口按照n%的速度进行变化，其中n的值在-10到+10间随机选取
		ve_city5[i].elevation=ve[i].elevation;	
		ve_city5[i].name=ve[i].name;
		for (int j=0;j<12;j++)          
		{
			int n=rand()%4;
			Enum_weather w1=(enum Enum_weather)(n);
			char* nc=weather_str(w1); 
			ve_city5[i].weather[j]=nc;
		}
	}
	FILE *fp5;
	fp5=fopen("D://City2013.txt","w+");
	fprintf(fp5,"年份   城市名称   城市人口   海拔高度   1月天气   2月天气    3月天气    4月天气    5月天气    6月天气    7月天气    8月天气    9月天气    10月天气   11月天气   12月天气\n");
	for ( i=0;i<1023;i++)
	{
		char name2[10]={0};
		for (int k=0;k<ve_city5[i].name.size();k++)    //string类型转char数组
		{
			name2[k]=ve_city5[i].name[k];
		}
		fprintf(fp5,"%d%10s%10d%10d",ve_city5[i].year,name2,ve_city5[i].persons,ve_city5[i].elevation);
		for (int j=0;j<12;j++)
		{
			fprintf(fp5,"%11s",ve_city5[i].weather[j]);
		}
		fprintf(fp5,"\n");
	}
	fclose(fp5);
	//2014年各城市数据获取，并将数据写入City2014.txt文件中
	for ( i=0;i<1023;i++)
	{
		ve_city6[i].year=2014;
		ve_city6[i].persons=(1+(rand()%20-10)*0.01)*ve[i].persons; //2014年各城市人口按照n%的速度进行变化，其中n的值在-10到+10间随机选取
		ve_city6[i].elevation=ve[i].elevation;	
		ve_city6[i].name=ve[i].name;
		for (int j=0;j<12;j++)          
		{
			int n=rand()%4;
			Enum_weather w1=(enum Enum_weather)(n);
			char* nc=weather_str(w1); 
			ve_city6[i].weather[j]=nc;
		}
	}
	FILE *fp6;
	fp6=fopen("D://City2014.txt","w+");
	fprintf(fp6,"年份   城市名称   城市人口   海拔高度   1月天气   2月天气    3月天气    4月天气    5月天气    6月天气    7月天气    8月天气    9月天气    10月天气   11月天气   12月天气\n");
	for ( i=0;i<1023;i++)
	{
		char name2[10]={0};
		for (int k=0;k<ve_city6[i].name.size();k++)    //string类型转char数组
		{
			name2[k]=ve_city6[i].name[k];
		}
		fprintf(fp6,"%d%10s%10d%10d",ve_city6[i].year,name2,ve_city6[i].persons,ve_city6[i].elevation);
		for (int j=0;j<12;j++)
		{
			fprintf(fp6,"%11s",ve_city6[i].weather[j]);
		}
		fprintf(fp6,"\n");
	}
	fclose(fp6);
	//2015年各城市数据获取，并将数据写入City2015.txt文件中
	for ( i=0;i<1023;i++)
	{
		ve_city7[i].year=2015;
		ve_city7[i].persons=(1+(rand()%20-10)*0.01)*ve[i].persons; //2015年各城市人口按照n%的速度进行变化，其中n的值在-10到+10间随机选取
		ve_city7[i].elevation=ve[i].elevation;	
		ve_city7[i].name=ve[i].name;
		for (int j=0;j<12;j++)          
		{
			int n=rand()%4;
			Enum_weather w1=(enum Enum_weather)(n);
			char* nc=weather_str(w1); 
			ve_city7[i].weather[j]=nc;
		}
	}
	FILE *fp7;
	fp7=fopen("D://City2015.txt","w+");
	fprintf(fp7,"年份   城市名称   城市人口   海拔高度   1月天气   2月天气    3月天气    4月天气    5月天气    6月天气    7月天气    8月天气    9月天气    10月天气   11月天气   12月天气\n");
	for ( i=0;i<1023;i++)
	{
		char name2[10]={0};
		for (int k=0;k<ve_city7[i].name.size();k++)    //string类型转char数组
		{
			name2[k]=ve_city7[i].name[k];
		}
		fprintf(fp7,"%d%10s%10d%10d",ve_city7[i].year,name2,ve_city7[i].persons,ve_city7[i].elevation);
		for (int j=0;j<12;j++)
		{
			fprintf(fp7,"%11s",ve_city7[i].weather[j]);
		}
		fprintf(fp7,"\n");
	}
	fclose(fp7);
	//2016年各城市数据获取，并将数据写入City2016.txt文件中
	for ( i=0;i<1023;i++)
	{
		ve_city8[i].year=2016;
		ve_city8[i].persons=(1+(rand()%20-10)*0.01)*ve[i].persons; //2016年各城市人口按照n%的速度进行变化，其中n的值在-10到+10间随机选取
		ve_city8[i].elevation=ve[i].elevation;	
		ve_city8[i].name=ve[i].name;
		for (int j=0;j<12;j++)          
		{
			int n=rand()%4;
			Enum_weather w1=(enum Enum_weather)(n);
			char* nc=weather_str(w1); 
			ve_city8[i].weather[j]=nc;
		}
	}
	FILE *fp8;
	fp8=fopen("D://City2016.txt","w+");
	fprintf(fp8,"年份   城市名称   城市人口   海拔高度   1月天气   2月天气    3月天气    4月天气    5月天气    6月天气    7月天气    8月天气    9月天气    10月天气   11月天气   12月天气\n");
	for ( i=0;i<1023;i++)
	{
		char name2[10]={0};
		for (int k=0;k<ve_city8[i].name.size();k++)    //string类型转char数组
		{
			name2[k]=ve_city8[i].name[k];
		}
		fprintf(fp8,"%d%10s%10d%10d",ve_city8[i].year,name2,ve_city8[i].persons,ve_city8[i].elevation);
		for (int j=0;j<12;j++)
		{
			fprintf(fp8,"%11s",ve_city8[i].weather[j]);
		}
		fprintf(fp8,"\n");
	}
	fclose(fp8);
	//2017年各城市数据获取，并将数据写入City2017.txt文件中
	for ( i=0;i<1023;i++)
	{
		ve_city9[i].year=2017;
		ve_city9[i].persons=(1+(rand()%20-10)*0.01)*ve[i].persons; //2017年各城市人口按照n%的速度进行变化，其中n的值在-10到+10间随机选取
		ve_city9[i].elevation=ve[i].elevation;	
		ve_city9[i].name=ve[i].name;
		for (int j=0;j<12;j++)          
		{
			int n=rand()%4;
			Enum_weather w1=(enum Enum_weather)(n);
			char* nc=weather_str(w1); 
			ve_city9[i].weather[j]=nc;
		}
	}
	FILE *fp9;
	fp9=fopen("D://City2017.txt","w+");
	fprintf(fp9,"年份   城市名称   城市人口   海拔高度   1月天气   2月天气    3月天气    4月天气    5月天气    6月天气    7月天气    8月天气    9月天气    10月天气   11月天气   12月天气\n");
	for ( i=0;i<1023;i++)
	{
		char name2[10]={0};
		for (int k=0;k<ve_city9[i].name.size();k++)    //string类型转char数组
		{
			name2[k]=ve_city9[i].name[k];
		}
		fprintf(fp9,"%d%10s%10d%10d",ve_city9[i].year,name2,ve_city9[i].persons,ve_city9[i].elevation);
		for (int j=0;j<12;j++)
		{
			fprintf(fp9,"%11s",ve_city9[i].weather[j]);
		}
		fprintf(fp9,"\n");
	}
	fclose(fp9);
	//2018年各城市数据获取，并将数据写入City2018.txt文件中
	for ( i=0;i<1023;i++)
	{
		ve_city10[i].year=2018;
		ve_city10[i].persons=(1+(rand()%20-10)*0.01)*ve[i].persons; //2018年各城市人口按照n%的速度进行变化，其中n的值在-10到+10间随机选取
		ve_city10[i].elevation=ve[i].elevation;	
		ve_city10[i].name=ve[i].name;
		for (int j=0;j<12;j++)          
		{
			int n=rand()%4;
			Enum_weather w1=(enum Enum_weather)(n);
			char* nc=weather_str(w1); 
			ve_city10[i].weather[j]=nc;
		}
	}
	FILE *fp10;
	fp10=fopen("D://City2018.txt","w+");
	fprintf(fp10,"年份   城市名称   城市人口   海拔高度   1月天气   2月天气    3月天气    4月天气    5月天气    6月天气    7月天气    8月天气    9月天气    10月天气   11月天气   12月天气\n");
	for ( i=0;i<1023;i++)
	{
		char name2[10]={0};
		for (int k=0;k<ve_city10[i].name.size();k++)    //string类型转char数组
		{
			name2[k]=ve_city10[i].name[k];
		}
		fprintf(fp10,"%d%10s%10d%10d",ve_city10[i].year,name2,ve_city10[i].persons,ve_city10[i].elevation);
		for (int j=0;j<12;j++)
		{
			fprintf(fp10,"%11s",ve_city10[i].weather[j]);
		}
		fprintf(fp10,"\n");
	}
	fclose(fp10);
	//分别将2009到2019年各城市的人口数存入下列命名的vector中
	vector<int>v1,v2,v3,v4,v5,v6,v7,v8,v9,v10;
	for ( i=0;i<1023;i++)                  //将2009-2019年各城市的人口数据分别存入
	{
		v1.push_back(ve_city1[i].persons); 
		v2.push_back(ve_city2[i].persons);
		v3.push_back(ve_city3[i].persons);
		v4.push_back(ve_city4[i].persons);
		v5.push_back(ve_city5[i].persons);
		v6.push_back(ve_city6[i].persons);
		v7.push_back(ve_city7[i].persons);
		v8.push_back(ve_city8[i].persons);
		v9.push_back(ve_city9[i].persons);
		v10.push_back(ve_city10[i].persons);
	}
	//对每年各城市的人口数进行排序，调用sort()函数，由小到大排序
	sort(v1.begin(),v1.end());
	sort(v2.begin(),v2.end());
	sort(v3.begin(),v3.end());
	sort(v4.begin(),v4.end());
	sort(v5.begin(),v5.end());
	sort(v6.begin(),v6.end());
	sort(v7.begin(),v7.end());
	sort(v8.begin(),v8.end());
	sort(v9.begin(),v9.end());
	sort(v10.begin(),v10.end());
	char much1[10]={0},little1[10]={0},centre1[10]={0},much2[10]={0},little2[10]={0},centre2[10]={0},much3[10]={0},little3[10]={0},centre3[10]={0},much4[10]={0},little4[10]={0},centre4[10]={0};
	char much5[10]={0},little5[10]={0},centre5[10]={0},much6[10]={0},little6[10]={0},centre6[10]={0},much7[10]={0},little7[10]={0},centre7[10]={0},much8[10]={0},little8[10]={0},centre8[10]={0};
	char much9[10]={0},little9[10]={0},centre9[10]={0},much10[10]={0},little10[10]={0},centre10[10]={0};
	//分别获得从2009~2019这十年里人数最多、人数最少、人数处于中位的城市的名称
	for ( i=0;i<1023;i++)                   
	{
		if (ve_city1[i].persons==v1[1022])  //2009年人口最多城市、最少城市、中等位城市
		{
			string s1=ve_city1[i].name;
			for (int k=0;k<s1.size();k++)
			{
				much1[k]=s1[k];
			}
		}
		if (ve_city1[i].persons==v1[0])
		{
			string s1=ve_city1[i].name;
			for (int k=0;k<s1.size();k++)
			{
				little1[k]=s1[k];
			}
		}
		if (ve_city1[i].persons==v1[511])
		{
			string s1=ve_city1[i].name;
			for (int k=0;k<s1.size();k++)
			{
				centre1[k]=s1[k];
			}
		}
		if (ve_city2[i].persons==v2[1022])   //2010年人口最多城市、最少城市、中等位城市
		{
			string s1=ve_city2[i].name;
			for (int k=0;k<s1.size();k++)
			{
				much2[k]=s1[k];
			}
		}
		if (ve_city2[i].persons==v2[0])
		{
			string s1=ve_city2[i].name;
			for (int k=0;k<s1.size();k++)
			{
				little2[k]=s1[k];
			}
		}
		if (ve_city2[i].persons==v2[511])
		{
			string s1=ve_city2[i].name;
			for (int k=0;k<s1.size();k++)
			{
				centre2[k]=s1[k];
			}
		}
		if (ve_city3[i].persons==v3[1022])   //2011年人口最多城市、最少城市、中等位城市
		{
			string s1=ve_city3[i].name;
			for (int k=0;k<s1.size();k++)
			{
				much3[k]=s1[k];
			}
		}
		if (ve_city3[i].persons==v3[0])
		{
			string s1=ve_city3[i].name;
			for (int k=0;k<s1.size();k++)
			{
				little3[k]=s1[k];
			}
		}
		if (ve_city3[i].persons==v3[511])
		{
			string s1=ve_city3[i].name;
			for (int k=0;k<s1.size();k++)
			{
				centre3[k]=s1[k];
			}
		}
		if (ve_city4[i].persons==v4[1022])   //2012年人口最多城市、最少城市、中等位城市
		{
			string s1=ve_city4[i].name;
			for (int k=0;k<s1.size();k++)
			{
				much4[k]=s1[k];
			}
		}
		if (ve_city4[i].persons==v4[0])
		{
			string s1=ve_city4[i].name;
			for (int k=0;k<s1.size();k++)
			{
				little4[k]=s1[k];
			}
		}
		if (ve_city4[i].persons==v4[511])
		{
			string s1=ve_city4[i].name;
			for (int k=0;k<s1.size();k++)
			{
				centre4[k]=s1[k];
			}
		}
		if (ve_city5[i].persons==v5[1022])   //2013年人口最多城市、最少城市、中等位城市
		{
			string s1=ve_city5[i].name;
			for (int k=0;k<s1.size();k++)
			{
				much5[k]=s1[k];
			}
		}
		if (ve_city5[i].persons==v5[0])
		{
			string s1=ve_city5[i].name;
			for (int k=0;k<s1.size();k++)
			{
				little5[k]=s1[k];
			}
		}
		if (ve_city5[i].persons==v5[511])
		{
			string s1=ve_city5[i].name;
			for (int k=0;k<s1.size();k++)
			{
				centre5[k]=s1[k];
			}
		}
		if (ve_city6[i].persons==v6[1022])   //2014年人口最多城市、最少城市、中等位城市
		{
			string s1=ve_city6[i].name;
			for (int k=0;k<s1.size();k++)
			{
				much6[k]=s1[k];
			}
		}
		if (ve_city6[i].persons==v6[0])
		{
			string s1=ve_city6[i].name;
			for (int k=0;k<s1.size();k++)
			{
				little6[k]=s1[k];
			}
		}
		if (ve_city6[i].persons==v6[511])
		{
			string s1=ve_city6[i].name;
			for (int k=0;k<s1.size();k++)
			{
				centre6[k]=s1[k];
			}
		}
		if (ve_city7[i].persons==v7[1022])   //2015年人口最多城市、最少城市、中等位城市
		{
			string s1=ve_city7[i].name;
			for (int k=0;k<s1.size();k++)
			{
				much7[k]=s1[k];
			}
		}
		if (ve_city7[i].persons==v7[0])
		{
			string s1=ve_city7[i].name;
			for (int k=0;k<s1.size();k++)
			{
				little7[k]=s1[k];
			}
		}
		if (ve_city7[i].persons==v7[511])
		{
			string s1=ve_city7[i].name;
			for (int k=0;k<s1.size();k++)
			{
				centre7[k]=s1[k];
			}
		}
		if (ve_city8[i].persons==v8[1022])   //2016年人口最多城市、最少城市、中等位城市
		{
			string s1=ve_city8[i].name;
			for (int k=0;k<s1.size();k++)
			{
				much8[k]=s1[k];
			}
		}
		if (ve_city8[i].persons==v8[0])
		{
			string s1=ve_city8[i].name;
			for (int k=0;k<s1.size();k++)
			{
				little8[k]=s1[k];
			}
		}
		if (ve_city8[i].persons==v8[511])
		{
			string s1=ve_city8[i].name;
			for (int k=0;k<s1.size();k++)
			{
				centre8[k]=s1[k];
			}
		}
		if (ve_city9[i].persons==v9[1022])   //2017年人口最多城市、最少城市、中等位城市
		{
			string s1=ve_city9[i].name;
			for (int k=0;k<s1.size();k++)
			{
				much9[k]=s1[k];
			}
		}
		if (ve_city9[i].persons==v9[0])
		{
			string s1=ve_city9[i].name;
			for (int k=0;k<s1.size();k++)
			{
				little9[k]=s1[k];
			}
		}
		if (ve_city9[i].persons==v9[511])
		{
			string s1=ve_city9[i].name;
			for (int k=0;k<s1.size();k++)
			{
				centre9[k]=s1[k];
			}
		}
		if (ve_city10[i].persons==v10[1022])   //2018年人口最多城市、最少城市、中等位城市
		{
			string s1=ve_city10[i].name;
			for (int k=0;k<s1.size();k++)
			{
				much10[k]=s1[k];
			}
		}
		if (ve_city10[i].persons==v10[0])
		{
			string s1=ve_city10[i].name;
			for (int k=0;k<s1.size();k++)
			{
				little10[k]=s1[k];
			}
		}
		if (ve_city10[i].persons==v10[511])
		{
			string s1=ve_city10[i].name;
			for (int k=0;k<s1.size();k++)
			{
				centre10[k]=s1[k];
			}
		}


	}
	//2009~2019这10年里人口最多、最少和人口处于中位数的各个城市，分别将数据写入Population.txt文件中
	FILE *fpp;
	fpp=fopen("D://Population.txt","w+");
	fprintf(fpp,"年份    最多人口城市名称   最少人口城市名称   中位数人口城市名称\n");
	fprintf(fpp,"2009   %10s%18s%22s\n",much1,little1,centre1);
	fprintf(fpp,"2010   %10s%18s%22s\n",much2,little2,centre2);
	fprintf(fpp,"2011   %10s%18s%22s\n",much3,little3,centre3);
	fprintf(fpp,"2012   %10s%18s%22s\n",much4,little4,centre4);
	fprintf(fpp,"2013   %10s%18s%22s\n",much5,little5,centre5);
	fprintf(fpp,"2014   %10s%18s%22s\n",much6,little6,centre6);
	fprintf(fpp,"2015   %10s%18s%22s\n",much7,little7,centre7);
	fprintf(fpp,"2016   %10s%18s%22s\n",much8,little8,centre8);
	fprintf(fpp,"2017   %10s%18s%22s\n",much9,little9,centre9);
	fprintf(fpp,"2018   %10s%18s%22s\n",much10,little10,centre10);
	fclose(fpp);
	//获取2009~2019年每年拥有最好天气的城市的数据信息
	vector<int>we1;
	int max1=0,max2=0,max3=0,max4=0,max5=0,max6=0,max7=0,max8=0,max9=0,max10=0;
	int n1=0,n2=0,n3=0,n4=0,n5=0,n6=0,n7=0,n8=0,n9=0,n10=0;
	string str1,str2,str3,str4,str5,str6,str7,str8,str9,str10;
	char weat1[10]={0},weat2[10]={0},weat3[10]={0},weat4[10]={0},weat5[10]={0},weat6[10]={0},weat7[10]={0},weat8[10]={0},weat9[10]={0},weat10[10]={0};
	for( i=0;i<1023;i++)     //2009年拥有天气最好的城市名称及最好天气数
	{
		int w=0;
		for (int j=0;j<12;j++)
		{
			if (ve_city1[i].weather[j]=="Sunny")
			{
				w++;
			}
		}
		if (w>max1)
		{
			max1=w;
			n1=i;
		}
	}
	str1=ve_city1[n1].name;
	for ( i=0;i<str1.size();i++)
	{
		weat1[i]=str1[i];
	}
	for( i=0;i<1023;i++)     //2010年拥有天气最好的城市名称及最好天气数
	{
		int w=0;
		for (int j=0;j<12;j++)
		{
			if (ve_city2[i].weather[j]=="Sunny")
			{
				w++;
			}
		}
		if (w>max2)
		{
			max2=w;
			n2=i;
		}
	}
	str2=ve_city2[n2].name;
	for ( i=0;i<str2.size();i++)
	{
		weat2[i]=str2[i];
	}
	for( i=0;i<1023;i++)     //2011年拥有天气最好的城市名称及最好天气数
	{
		int w=0;
		for (int j=0;j<12;j++)
		{
			if (ve_city3[i].weather[j]=="Sunny")
			{
				w++;
			}
		}
		if (w>max3)
		{
			max3=w;
			n3=i;
		}
	}
	str3=ve_city3[n3].name;
	for ( i=0;i<str3.size();i++)
	{
		weat3[i]=str3[i];
	}
	for( i=0;i<1023;i++)     //2012年拥有天气最好的城市名称及最好天气数
	{
		int w=0;
		for (int j=0;j<12;j++)
		{
			if (ve_city4[i].weather[j]=="Sunny")
			{
				w++;
			}
		}
		if (w>max4)
		{
			max4=w;
			n4=i;
		}
	}
	str4=ve_city4[n4].name;
	for ( i=0;i<str4.size();i++)
	{
		weat4[i]=str4[i];
	}
	for( i=0;i<1023;i++)     //2013年拥有天气最好的城市名称及最好天气数
	{
		int w=0;
		for (int j=0;j<12;j++)
		{
			if (ve_city5[i].weather[j]=="Sunny")
			{
				w++;
			}
		}
		if (w>max5)
		{
			max5=w;
			n5=i;
		}
	}
	str5=ve_city5[n5].name;
	for ( i=0;i<str5.size();i++)
	{
		weat5[i]=str5[i];
	}
	for( i=0;i<1023;i++)     //2014年拥有天气最好的城市名称及最好天气数
	{
		int w=0;
		for (int j=0;j<12;j++)
		{
			if (ve_city6[i].weather[j]=="Sunny")
			{
				w++;
			}
		}
		if (w>max6)
		{
			max6=w;
			n6=i;
		}
	}
	str6=ve_city6[n6].name;
	for ( i=0;i<str6.size();i++)
	{
		weat6[i]=str6[i];
	}
	for( i=0;i<1023;i++)     //2015年拥有天气最好的城市名称及最好天气数
	{
		int w=0;
		for (int j=0;j<12;j++)
		{
			if (ve_city7[i].weather[j]=="Sunny")
			{
				w++;
			}
		}
		if (w>max7)
		{
			max7=w;
			n7=i;
		}
	}
	str7=ve_city7[n7].name;
	for ( i=0;i<str7.size();i++)
	{
		weat7[i]=str7[i];
	}
	for( i=0;i<1023;i++)     //2016年拥有天气最好的城市名称及最好天气数
	{
		int w=0;
		for (int j=0;j<12;j++)
		{
			if (ve_city8[i].weather[j]=="Sunny")
			{
				w++;
			}
		}
		if (w>max8)
		{
			max8=w;
			n8=i;
		}
	}
	str8=ve_city8[n8].name;
	for ( i=0;i<str8.size();i++)
	{
		weat8[i]=str8[i];
	}
	for( i=0;i<1023;i++)     //2017年拥有天气最好的城市名称及最好天气数
	{
		int w=0;
		for (int j=0;j<12;j++)
		{
			if (ve_city9[i].weather[j]=="Sunny")
			{
				w++;
			}
		}
		if (w>max9)
		{
			max9=w;
			n9=i;
		}
	}
	str9=ve_city9[n9].name;
	for ( i=0;i<str9.size();i++)
	{
		weat9[i]=str9[i];
	}
	for( i=0;i<1023;i++)     //2018年拥有天气最好的城市名称及最好天气数
	{
		int w=0;
		for (int j=0;j<12;j++)
		{
			if (ve_city10[i].weather[j]=="Sunny")
			{
				w++;
			}
		}
		if (w>max10)
		{
			max10=w;
			n10=i;
		}
	}
	str10=ve_city10[n10].name;
	for ( i=0;i<str10.size();i++)
	{
		weat10[i]=str10[i];
	}
	//2009年到2019年的10年间，每年拥有最好天气数量的城市名称及最好天气数写入Weather.txt文件中
	FILE *fp_we;
	fp_we=fopen("D://Weather.txt","w+");
	fprintf(fp_we,"年份     城市名称     Sunny数量\n");
	fprintf(fp_we,"2009%12s%10d\n",weat1,max1);
	fprintf(fp_we,"2010%12s%10d\n",weat2,max2);
	fprintf(fp_we,"2011%12s%10d\n",weat3,max3);
	fprintf(fp_we,"2012%12s%10d\n",weat4,max4);
	fprintf(fp_we,"2013%12s%10d\n",weat5,max5);
	fprintf(fp_we,"2014%12s%10d\n",weat6,max6);
	fprintf(fp_we,"2015%12s%10d\n",weat7,max7);
	fprintf(fp_we,"2016%12s%10d\n",weat8,max8);
	fprintf(fp_we,"2017%12s%10d\n",weat9,max9);
	fprintf(fp_we,"2018%12s%10d\n",weat10,max10);
	fclose(fp_we);
	//找出所有海拔在1000-3500米的城市，结果写入文件altitude.txt
	vector<string>ss;
	for ( i=0;i<1023;i++)
	{
		if ((ve[i].elevation>=1000)&&(ve[i].elevation<=3500))
		{
			string s=ve[i].name;
			ss.push_back(s);
		}
	}
	FILE *fp_high;
	fp_high=fopen("D://altitude.txt","w+");
	fprintf(fp_high,"所有海拔在1000-3500米的城市如下\n");
	for ( i=0;i<ss.size();i++)
	{
		char ch[10]={0};
		for (int j=0;j<ss[i].size();j++)
		{
			ch[j]=ss[i][j];
		}
		fprintf(fp_high,"%10s\n",ch);
	}
	fclose(fp_high);
	vector<int>ve_sort1;
	vector<int>ve_sort2;
	for ( i=0;i<ve.size();i++)  //将各城市的海拔写入ve_sort1,ve.sort2容器中
	{
		ve_sort1.push_back(ve[i].elevation);
		ve_sort2.push_back(ve[i].elevation);
	}
	sort(ve_sort1.begin(),ve_sort1.end());//对各城市的海拔由小到大进行排序（升序）
	sort(ve_sort2.begin(),ve_sort2.end());
	reverse(ve_sort2.begin(),ve_sort2.end());// 对各城市的海拔由大到小进行排序（降序）
	vector<City>ve_airport(1023);
	vector<City>ve_airport1(1023);
	vector<City>ve_airport2(1023);
 	for ( i=0, j=0;(i<1023)&&(j<1023);i++)            //选取海拔在0~2500的城市，并存入ve_airport中
 	{
 		if ((ve[i].elevation>0)&&(ve[i].elevation<2500))
 		{
 			ve_airport[j].name=ve[i].name;
 			ve_airport[j].elevation=ve[i].elevation;
			ve_airport1[j].name=ve[i].name;
			ve_airport1[j].elevation=ve[i].elevation;
 			j++;
 		}
 	}
	for ( i=0;i<ve_airport.size();i++)
	{
		for (int j=0;j<ve_airport1.size();j++)
		{
			if (abs(ve_airport[i].elevation-ve_airport1[j].elevation)!=100)
			{
				ve_airport2[j].name=ve_airport1[j].name;
			}
		}
	}
	string s;
	//将选择城市结果输出Airport.txt文本文件中

}
```



