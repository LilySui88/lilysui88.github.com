---
layout: post
title: "学生成绩管理，C语言版"
description: ""
category: coding
tags: [c]
---
{% include JB/setup %}

## 事件的起因


一如往常，早上上班来到办公室，检查了昨天的数据报表，都按时出来了。因为最近手头的活都干的差不多了，心中窃喜，想着今天可以自己干点喜欢的事情。

But，突然接到大侄子的微信，

![大侄子的求救微信](http://upload-images.jianshu.io/upload_images/3367144-9dace2ad04684875.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

大侄子今年刚上大一，这不期末了么，老师布置了C语言的作业，具体要求就是如上图了~

孩子找我帮忙，他的第一反应就是让我写完，其实我知道这个如果我写当然不难，可是我都写了，大侄儿你自己怎么能理解更好呢？

为了让他能够好好写，我直接断了他的念想~

![必须告诉他自己写](http://upload-images.jianshu.io/upload_images/3367144-96f7875958f97951.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

大家以为事情到这就没了么？当然不~~~为了能更好的和侄子装X，我当然得提前把程序写好啦 😁😁😁

倒腾了一下午，写了几百行~万一那孩子最后写不出来，我肯定还是要救火的呀~~

想想，好多年不写C，距离大一C语言课设已经10年了。想当初我的课设也是抄的呀，不然我咋那么理解大侄儿呢，哈哈哈~

为了救救其他孩子们，把代码贴出来撒~

{% highlight c%}
#include<stdio.h>
#include<stdlib.h>

//结构体可以存放的学生信息最大个数，不可变变量
int const MAX_LENGTH=100;

//学生信息结构体数组，最多可以存放100个学生信息
struct student{
int id;  //学号
char *name;  //姓名
int age;  //年龄
float c_score;  //C语言成绩
float english_score;  //英语成绩
float database_score;  //数据库成绩
float total_score;	//总分
}student_array[MAX_LENGTH];

//学生信息数量
int student_count=0;

//函数声明
void print_all_students();
void input_info();
void query_info();
void modify_info();
void delete_info();
void compute_total_score();
void sort_info();
int search_one_student();
void print_one_student(struct student one);
void delete_one_student(int student_index);
char * get_item_name(int item_index);
void modify_one_student(int student_index);
void sort_by_id();
void sort_by_c_score();
void sort_by_english_score();
void sort_by_database_score();


//主函数
int main(){
	while(1){
		printf("请选择要使用的功能：\n");
		printf("录入信息，请输入1，然后回车！\n");
		printf("查询信息，请输入2，然后回车！\n");
		printf("修改信息，请输入3，然后回车！\n");
		printf("删除信息，请输入4，然后回车！\n");
		printf("计算总分，请输入5，然后回车！\n");
		printf("排序信息，请输入6，然后回车！\n");
		printf("输出全部，请输入0，然后回车！\n");
		printf("退出程序，请输入-1，然后回车！\n");
		//函数选择变量
		int function=0;
		//输入选择的函数编号值
		scanf("%d",&function);
		//根据输入的函数编号，执行对应的功能
		switch(function){
			case -1:
			exit(1);
			case 0:
			print_all_students();
			break;
			case 1:
			input_info();
			break;
			case 2:
			query_info();
			break;
			case 3:
			modify_info();
			break;
			case 4:
			delete_info();
			break;
			case 5:
			compute_total_score();
			break;
			case 6:
			sort_info();
			break;
			default:
			printf("请输入正确的功能编号！！！\n\n");
			break;
		}
	}
	return 0;
}


//录入信息函数
void input_info(){
	printf("当前功能————录入信息！\n");
	//判断是否还有空间
	if(student_count<MAX_LENGTH){
	//声明一些临时变量
		int id=0;
		char *name=(char *)malloc(100);
		int age=0;
		float c_score=0;
		float english_score=0;
		float database_score=0;
		printf("请输入学生信息，格式为：学号,姓名,年龄,C语言成绩,英语成绩,数据库成绩\n");
		scanf("%d %s %d %f %f %f",&id,name,&age,&c_score,&english_score,&database_score);
		printf("学生信息校对：学号：%d,姓名：%s,年龄：%d,C语言成绩:%f,英语成绩：%f,数据库成绩：%f\n",id,name,age,c_score,english_score,database_score);
	//学生信息加入结构体数组
		student_array[student_count].id=id;
		student_array[student_count].name=name;
		student_array[student_count].age=age;
		student_array[student_count].c_score=c_score;
		student_array[student_count].english_score=english_score;
		student_array[student_count].database_score=database_score;
		student_count++;
	//是否继续录入信息
		printf("是否继续录入信息？继续请输入y,返回主菜单输入n\n");
		char go_on;
		scanf("%s",&go_on);
		if(go_on=='y'){
			input_info();
		}
	}else{
		printf("学生结构体数据已满，可以删除一部分学生信息！\n");
	}
}

//查询信息函数
void query_info(){
	printf("当前功能————查询信息！\n");
	printf("请输入学生的学号\n");
	int id=0;
	scanf("%d",&id);
	//查找输入id学生的序号
	int student_index=search_one_student(id);
	if(student_index!=-1){
		print_one_student(student_array[student_index]);
	}else{
		printf("没有该学号的学生！\n");
	}
	//是否继续查询信息
	printf("是否继续查询信息？继续请输入y,返回主菜单输入n\n");
	char go_on;
	scanf("%s",&go_on);
	if(go_on=='y')
		query_info();
}

//修改信息函数
void modify_info(){
	printf("当前功能————修改信息！\n");
	printf("请输入学生的学号\n");
	int id=0;
	scanf("%d",&id);
	//查找输入id学生的序号
	int student_index=search_one_student(id);
	if(student_index!=-1){
		modify_one_student(student_index);
	}else{
		printf("没有该学号的学生！\n");
	}


}

//删除信息函数
void delete_info(){
	printf("当前功能————删除信息！\n");
	printf("请输入学生的学号\n");
	int id=0;
	scanf("%d",&id);
	//查找输入id学生的序号
	int student_index=search_one_student(id);
	if(student_index!=-1){
		//防止student_index被改变，传入temp_index计算
		int temp_index=student_index;
		print_one_student(student_array[temp_index]);
		//删除前进行确认
		printf("确定删除学号 %d 同学的信息？继续请输入y\n",id);
		char be_true;
		scanf("%s",&be_true);
		if(be_true=='y'){
			printf("%d\n", student_index);
			//执行删除动作
			delete_one_student(student_index);
		}
	}else{
		printf("没有该学号的学生！\n");
	}
	//是否继续删除信息
	printf("是否继续删除信息？继续请输入y,返回主菜单输入n\n");
	char go_on;
	scanf("%s",&go_on);
	if(go_on=='y')
		delete_info();
}

//计算总分函数
void compute_total_score(){
	printf("当前功能————计算总分！\n");
	for (int i = 0; i < student_count; ++i)
	{
		student_array[i].total_score=student_array[i].c_score+student_array[i].english_score+student_array[i].database_score;
		print_one_student(student_array[i]);
		printf("总成绩：%f\n", student_array[i].total_score);
	}
	printf("总分计算完成！！！\n");
}

//成绩排序函数
void sort_info(){
	printf("当前功能————成绩排序！\n");
	printf("排序前所有学生信息如下：\n");
	print_all_students();
	int sort_type;
	while(1){
		printf("请输入排序字段，学号：1,C语言成绩:2,英语成绩：3,数据库成绩：4\n");
		scanf("%d",&sort_type);
		if(sort_type>=1&&sort_type<=4)
			break;
	}
	switch(sort_type){
		case 1:
		sort_by_id();
		break;
		case 2:
		sort_by_c_score();
		break;
		case 3:
		sort_by_english_score();
		break;
		case 4:
		sort_by_database_score();
		break;
	}
	printf("排序后所有学生信息如下：\n");
	print_all_students();	
	//是否继续删除信息
	printf("是否继续排序信息？继续请输入y,返回主菜单输入n\n");
	char go_on;
	scanf("%s",&go_on);
	if(go_on=='y')
		sort_info();
}

//根据输入的学号，遍历结构体数组，若存在该学生，返回数组下标，不存在返回-1
int search_one_student(int id){
	for (int i = 0; i < student_count; ++i)
	{
		if(student_array[i].id==id){
			return i;
		}
	}
	return -1;
}

//输出某个学生的信息
void print_one_student(struct student one){
	printf("学生信息：学号：%d,姓名：%s,年龄：%d,C语言成绩:%f,英语成绩：%f,数据库成绩：%f\n",one.id,one.name,one.age,one.c_score,one.english_score,one.database_score);	
}


//输出所有学生的信息
void print_all_students(){
	if(student_count==0){
		printf("暂无学生信息\n\n\n");
	}
	for (int i = 0; i < student_count; ++i)
	{
		print_one_student(student_array[i]);
	}
}

void modify_one_student(int student_index){
	//修改前，输出学生信息
	print_one_student(student_array[student_index]);
	//字段序号初始值
	int item_index=0;
	//不允许修改学号字段
	while(1){
		printf("请输入要修改的字段序号，姓名：1,年龄：2,C语言成绩:3,英语成绩：4,数据库成绩：5\n");
		scanf("%d",&item_index);
		if(item_index>=1&&item_index<=5)
			break;
	}
	switch(item_index){
		case 1:
		printf("请输入 %s 字段的新值\n", get_item_name(item_index));
		char* item_value_1=(char *)malloc(100);;
		scanf("%s",item_value_1);		
		student_array[student_index].name=item_value_1;
		break;
		case 2:
		printf("请输入 %s 字段的新值\n", get_item_name(item_index));
		int item_value_2;
		scanf("%d",&item_value_2);	
		student_array[student_index].age=item_value_2;
		break;
		case 3:
		printf("请输入 %s 字段的新值\n", get_item_name(item_index));
		float item_value_3;
		scanf("%f",&item_value_3);	
		student_array[student_index].c_score=item_value_3;
		break;
		case 4:
		printf("请输入 %s 字段的新值\n", get_item_name(item_index));
		float item_value_4;
		scanf("%f",&item_value_4);	
		student_array[student_index].english_score=item_value_4;
		break;
		case 5:
		printf("请输入 %s 字段的新值\n", get_item_name(item_index));
		float item_value_5;
		scanf("%f",&item_value_5);	
		student_array[student_index].database_score=item_value_5;
		break;
	}
	printf("修改成功！新的学生信息如下：\n");
		//修改后输出学生信息
	print_one_student(student_array[student_index]);	
	//是否继续删除信息
	printf("是否继续修改该学生信息？继续请输入y,返回主菜单输入n\n");
	char go_on;
	scanf("%s",&go_on);
	if(go_on=='y')
		modify_one_student(student_index);
}

//删除数组对应序号的学生信息，把i位置后面的数据组元素向前移动覆盖i，student_count计数器减1
void delete_one_student(int student_index){
	for (int i = student_index; i < student_count-1; ++i)
	{
		student_array[i]=student_array[i+1];
	}
	student_count--;
	printf("删除完成\n\n\n");
}

//根据输入的字段序号，返回字段名称
char * get_item_name(int item_index){
	switch(item_index){
		case 0:
		return "学号";
		case 1:
		return "姓名";
		case 2:
		return "年龄";
		case 3:
		return "C语言成绩";
		case 4:
		return "英语成绩";
		case 5:
		return "数据库成绩";
		default:
		return "";
	}
}

//按照id排序，使用最简单的冒泡排序法
void sort_by_id(){
	for (int i = 0; i < student_count; ++i)
	{
		for (int j = i; j < student_count; ++j)
		{
			if(student_array[i].id>student_array[j].id){
				struct student temp=student_array[i];
				student_array[i]=student_array[j];
				student_array[j]=temp;
			}
		}
	}
	printf("按照 学号 排序完成\n");
}

//按照C语言成绩排序，使用最简单的冒泡排序法
void sort_by_c_score(){
	for (int i = 0; i < student_count; ++i)
	{
		for (int j = i; j < student_count; ++j)
		{
			if(student_array[i].c_score>student_array[j].c_score){
				struct student temp=student_array[i];
				student_array[i]=student_array[j];
				student_array[j]=temp;
			}
		}
	}
	printf("按照 C语言成绩 排序完成\n");
}

//按照英语成绩排序，使用最简单的冒泡排序法
void sort_by_english_score(){
	for (int i = 0; i < student_count; ++i)
	{
		for (int j = i; j < student_count; ++j)
		{
			if(student_array[i].english_score>student_array[j].english_score){
				struct student temp=student_array[i];
				student_array[i]=student_array[j];
				student_array[j]=temp;
			}
		}
	}
	printf("按照 英语成绩 排序完成\n");
}

//按照数据库成绩排序，使用最简单的冒泡排序法
void sort_by_database_score(){
	for (int i = 0; i < student_count; ++i)
	{
		for (int j = i; j < student_count; ++j)
		{
			if(student_array[i].database_score>student_array[j].database_score){
				struct student temp=student_array[i];
				student_array[i]=student_array[j];
				student_array[j]=temp;
			}
		}
	}
	printf("按照 数据库成绩 排序完成\n");
}
{% endhighlight %}


