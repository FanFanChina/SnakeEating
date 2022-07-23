# 贪吃蛇

## 运行条件

### 需要安装 [EasyX](https://easyx.cn/)

**方式一：[EasyX](https://easyx.cn/)**
**方式二：见运行附件**

### VC++6.0 运行需要设置多线程

**方法：Project->Settings->C/C+±>Code Generation->Use run-time libray->Debug Multithread，或 Multithread，或 Debug Multithread DLL， 或 Multithread DLL 都可以**

### VS 2019 运行需要使用多字节字符集

**方法：项目 -> 属性 -> 高级 -> 字符集 -> 使用多字节字符集**

### Music 和 Picture 文件需要放在对应目录下

**[运行附件（百度云）](https://pan.baidu.com/s/1ue9ZXmyYwIoZz9z0Dl27Ng) 提取码：`lsd7`**
**[运行附件（阿里云）](https://www.aliyundrive.com/s/pk3Rnd2io9Q) 提取码：`7lpl`**

------

## 更新说明

### 引入多线程

### 加入感谢名单

### BGM—GET 微调

### 吃 food 时增加分数显示

### 开始时自动切换为英文输入法问题

### 修复 game over 后需要按两次 enter 键

### 引导画面文字播报与 BGM 踩点，缩短时间

### 修复进入游戏后连敲三下 Enter 的闪退问题

------

## 完整代码 (VC++6.0)

```c
C
#include<math.h>
#include<time.h>
#include<conio.h>
#include<stdio.h>
#include<stdlib.h>
#include<process.h>
#include<windows.h>
#include<graphics.h>
#include <mmsystem.h>
#pragma comment(lib, "winmm.lib")

#define lx 640
#define ly 480
#define Sup 7//换肤模式突变点
#define MAX 500

int key,thread,flag, mark, p, ans, i = 1, threadend = 1;; char s[1000]; double speed = 80;

enum DIR//4个方向,enum为枚举类型
{
	PAUS,
	UP,
	DOWN,
	LEFT,
	RIGHT,
};
struct Food
{
	int x;
	int y;
	int r;//半径
	bool flag;
	DWORD color;
}food;
struct Snake
{
	int size;
	int speed;
	int dir;
	DWORD color1;
	DWORD color2;
	POINT coor[MAX];
}snake;


void BGM_ALL();										//全局BGM		
unsigned __stdcall BGM_GET(void* pArguments);		//吃食物BGM		
void BGM_FAIL();									//失败BGM		
void Duce();										//引导菜单		
void Startmenu();									//开始菜单		
void Overmenu();									//结束菜单		
void Snakeinit();									//蛇初始化		
void Foodinit();		    						//食物初始化  		
void Init();										//初始化		
void Drawframe();									//画边框		
void Skinsystem(int i);								//皮肤系统		
void Drawsnake();									//普通切换皮肤模式		
void SupDrawsnake();								//快速切换皮肤模式		
void Drawfood();									//画食物		
void Draw();										//绘图		
void Firstcontrol();								//按键交互		
void Keycontrol();									//首次控制		
void Snakemove();									//蛇移动		
void Eatfood();										//吃食物
void Generatefood();								//生成食物
int Snakebump();									//蛇撞墙		
int Eatself();										//咬自己		
void Print();										//结束输出		
void Reset();										//重置		
void Thank();										//内测感谢名录		
void Shift();										//按下Shift键

void main()
{
	//多线程开始
	HANDLE   hThread;
	unsigned   threadID;
	hThread = (HANDLE)_beginthreadex(NULL, 0, &BGM_GET, NULL, 0, &threadID);
	BGM_ALL();
	Init();
	Startmenu();
	if (!key)//第一次开始按下shift键
	{
		Shift();
		key = 1;
	}
	while (1)
	{
		Sleep(long(speed));
		Draw();
		if (flag == 0)
		{
			Firstcontrol();
			flag = 1;
		}
		Snakemove();
		if (Eatself() || Snakebump())
		{
			mciSendString("close BGM1", 0, 0, 0);
			BGM_FAIL();
			break;
		}
		Eatfood();
	}
	Overmenu();
	switch (_getch())
	{
	case 13:
	{
		i = 1;
		speed = 80;
		flag = mark = ans = 0;
		Reset();
		threadend = 0;
		WaitForSingleObject(hThread, INFINITE);
		CloseHandle(hThread);
		threadend = 1;
		main();
	}
	break;
	case 27:
		Thank();
		closegraph();
		Print();
		break;
	}
	//多线程结束
	WaitForSingleObject(hThread, INFINITE);
	CloseHandle(hThread);
}

void BGM_ALL()
{
	mciSendString("open ./Music/Scheming_Weasel.mp3 alias BGM1", 0, 0, 0);
	mciSendString("play BGM1 repeat", 0, 0, 0);
}
unsigned __stdcall BGM_GET(void* pArguments)
{
	while (1)
	{
		if (thread)
		{
			mciSendString("open ./Music/Get.wav alias BGM2", 0, 0, 0);
			mciSendString("play BGM2 wait", 0, 0, 0);
			mciSendString("close BGM2 ", 0, 0, 0);
			thread = 0;
		}
		if (!threadend)
			break;
	}
	return 0;
}
void BGM_FAIL()
{
	mciSendString("open ./Music/Gameover.wav alias BGM3", 0, 0, 0);
	mciSendString("play BGM3 wait", 0, 0, 0);
	mciSendString("close BGM3 ", 0, 0, 0);
}
void Duce()
{
	if (p == 0) {
		p = 1;
		Sleep(300);
		IMAGE one;//定义一个图片名.
		loadimage(&one, "Picture\\01.png", lx, ly, 1);//从图片文件获取图像
		putimage(0, 0, &one);//绘制图像到屏幕，图片左上角坐标为(0,0)
		Sleep(1500);
		IMAGE two;//定义一个图片名.
		loadimage(&two, "Picture\\02.png", lx, ly, 1);//从图片文件获取图像
		putimage(0, 0, &two);//绘制图像到屏幕，图片左上角坐标为(0,0)
		Sleep(1500);
		IMAGE three;
		loadimage(&three, "Picture\\03.png", lx, ly, 1);//从图片文件获取图像
		putimage(0, 0, &three);//绘制图像到屏幕，图片左上角坐标为(0,0)
		Sleep(1400);
		IMAGE four;
		loadimage(&four, "Picture\\04.png", lx, ly, 1);//从图片文件获取图像
		putimage(0, 0, &four);//绘制图像到屏幕，图片左上角坐标为(0,0)
		Sleep(1500);
		IMAGE five;
		loadimage(&five, "Picture\\05.png", lx, ly, 1);//从图片文件获取图像
		putimage(0, 0, &five);//绘制图像到屏幕，图片左上角坐标为(0,0)
		Sleep(1400);
		cleardevice();
	}
}
void Startmenu()
{
	Duce();
	while (_kbhit())
		getchar();
	IMAGE welcome;
	loadimage(&welcome, "Picture\\00.png", lx, ly, 1);//从图片文件获取图像
	putimage(0, 0, &welcome);//绘制图像到屏幕，图片左上角坐标为(0,0)
	while (1)
	{
		if (_getch() == 13)
			break;
	}
}
void Snakeinit()
{
	srand(GetTickCount());
	snake.color1 = RGB(0, 235, 229);
	snake.size = 4;
	snake.speed = 10;
	snake.dir = PAUS;
	int randomx = rand() % (lx - 100) + 50;
	int randomy = rand() % (ly - 100) + 50;
	for (int i = 0; i <= snake.size - 1; i++)
	{
		snake.coor[i].x = (randomx / 10) * 10 - 10 * i;	//（除10乘10类方格化处理）
		snake.coor[i].y = (randomy / 10) * 10;
	}
}
void Foodinit()
{
	int j;
	food.flag = 0;
	while (!food.flag)
	{
		Generatefood();
		for (j = 0; j < snake.size; j++)
			if (fabs(snake.coor[j].x - food.x) <= food.r && fabs(snake.coor[j].y - food.y) <= food.r)
				Generatefood();
		food.flag = true;
	}
}
void Init()
{
	initgraph(lx, ly);//初始化窗口，大小640*480,/*SHOWCONSOLE*/
	Snakeinit();//蛇初始化
	Foodinit();//食物初始化
}
void Drawframe()
{
	setlinestyle(PS_SOLID, 20);
	setlinecolor(RGB(93, 107, 153));
	rectangle(0, 0, 640, 480);
}
void Skinsystem(int q)
{
	int i;
	switch (q)
	{
	case 1://YES
		//蛇头
		setfillcolor(RGB(93, 107, 153));
		solidcircle(snake.coor[0].x, snake.coor[0].y, 6);
		//蛇身
		setfillcolor(RGB(150, 157, 177));
		for (i = 1; i <= snake.size - 1; i++)
			solidcircle(snake.coor[i].x, snake.coor[i].y, 5);
		break;
	case 2://YES
		//蛇头
		setfillcolor(RGB(186, 63, 110));
		solidcircle(snake.coor[0].x, snake.coor[0].y, 6);
		//蛇身
		setfillcolor(RGB(248, 237, 203));
		for (i = 1; i <= snake.size - 1; i++)
			solidcircle(snake.coor[i].x, snake.coor[i].y, 5);
		break;
	case 3://YES
		setfillcolor(RGB(127, 205, 238));
		solidcircle(snake.coor[0].x, snake.coor[0].y, 6);
		setfillcolor(RGB(244, 241, 236));
		for (i = 1; i <= snake.size - 1; i++)
			solidcircle(snake.coor[i].x, snake.coor[i].y, 5);
		break;
	case 4://YES
		//蛇头
		setfillcolor(RGB(221, 192, 179));
		solidcircle(snake.coor[0].x, snake.coor[0].y, 6);
		//蛇身
		setfillcolor(RGB(244, 241, 236));
		for (i = 1; i <= snake.size - 1; i++)
			solidcircle(snake.coor[i].x, snake.coor[i].y, 5);
		break;
	case 5://YES
		setfillcolor(RGB(113, 111, 114));
		solidcircle(snake.coor[0].x, snake.coor[0].y, 6);
		setfillcolor(RGB(244, 241, 236));
		for (i = 1; i <= snake.size - 1; i++)
			solidcircle(snake.coor[i].x, snake.coor[i].y, 5);
		break;
		//蛇头
	case 6://YES
		setfillcolor(RGB(206, 124, 128));
		solidcircle(snake.coor[0].x, snake.coor[0].y, 6);
		//蛇身
		setfillcolor(RGB(170, 175, 231));
		for (i = 1; i <= snake.size - 1; i++)
			solidcircle(snake.coor[i].x, snake.coor[i].y, 5);
		break;
	case 7://YES
		setfillcolor(RGB(223, 165, 161));
		solidcircle(snake.coor[0].x, snake.coor[0].y, 6);
		//蛇身
		setfillcolor(RGB(123, 130, 184));
		for (i = 1; i <= snake.size - 1; i++)
			solidcircle(snake.coor[i].x, snake.coor[i].y, 5);
		break;
	}
}
void SupDrawsnake()
{
	if (i <= 7)
	{
		Skinsystem(i);
		i++;
	}
	else
	{
		Skinsystem(1);
		i = 1;
	}
}
void Drawsnake()
{
	if (i <= 7)
		Skinsystem(i);
	else
	{
		Skinsystem(1);
		i = 1;
	}
}
void Drawfood()
{
	srand(GetTickCount());
	if (food.flag)
	{
		setfillcolor(RGB(rand() % 256, rand() % 256, rand() % 256));
		solidcircle(food.x, food.y, food.r);
	}
}
void Draw()
{
	cleardevice();
	BeginBatchDraw();//双缓冲绘图
	setbkcolor(RGB(204, 213, 240));//背景色
	cleardevice();//清空绘图设备
	Drawframe();//画边框
	//画蛇(两种模式)
	if (ans % Sup == 0 && ans >= 1)
		SupDrawsnake();
	else
		Drawsnake();
	Drawfood();//画食物
	EndBatchDraw();//双缓冲绘图结束
}
void Firstcontrol()
{
	switch (_getch())
	{
	case 'W':
	case'w':
	case 72:
		if (snake.dir != DOWN)
			snake.dir = UP;
		break;
	case 'S':
	case 's':
	case 80:
		if (snake.dir != UP)
			snake.dir = DOWN;
		break;
	case 'A':
	case 'a':
	case 75:
		if (snake.dir != RIGHT)
			snake.dir = LEFT;
		break;
	case 'D':
	case 'd':
	case 77:
		if (snake.dir != LEFT)
			snake.dir = RIGHT;
		break;
	case 13:
		Firstcontrol();
		break;
	}
}
void Keycontrol()//按键控制
{
	if (_kbhit())
	{

		char key = _getch();
		//72,80,75,77
		switch (key)
		{
		case 'W'://上
		case'w':
		case 72:
			if (snake.dir != DOWN)
				snake.dir = UP;
			break;
		case 'S'://下
		case 's':
		case 80:
			if (snake.dir != UP)
				snake.dir = DOWN;
			break;
		case 'A'://左
		case 'a':
		case 75:
			if (snake.dir != RIGHT)
				snake.dir = LEFT;
			break;
		case 'D'://右
		case 'd':
		case 77:
			if (snake.dir != LEFT)
				snake.dir = RIGHT;
			break;
		case 13:
			getchar();
			break;
		}
	}
}
void Snakemove()
{
	int i;
	for (i = snake.size - 1; i > 0; i--)
		snake.coor[i] = snake.coor[i - 1];
	Keycontrol();
	switch (snake.dir)
	{
	case UP:
		snake.coor[0].y -= snake.speed;
		break;
	case DOWN:
		snake.coor[0].y += snake.speed;
		break;
	case LEFT:
		snake.coor[0].x -= snake.speed;
		break;
	case RIGHT:
		snake.coor[0].x += snake.speed;
		break;
	case PAUS:
		getchar();
		break;
	}
}
void Eatfood()
{
	if (food.flag && fabs(snake.coor[0].x - food.x) <= food.r && fabs(snake.coor[0].y - food.y) <= food.r)
	{
		thread = 1;//多线程开始执行
		i++;
		ans++;
		snake.color1 = food.color;
		if (speed > 60)
			speed -= 0.05;
		food.flag = false;
		mark += food.r;
		snake.size += food.r*2/3;
		//显示分数
		settextstyle(26, 0, "微软雅黑");
		settextcolor(COLORREF RGB(0, 0, 0));
		sprintf(s, "%d", mark);
		outtextxy(food.x, food.y, s);
	}
	int j;
	while (!food.flag)
	{
		Generatefood();
		for (j = 0; j < snake.size; j++)
			if (fabs(snake.coor[j].x - food.x) <= food.r && fabs(snake.coor[j].y - food.y) <= food.r)
				Generatefood();
		food.flag = true;
	}
}
void Generatefood()
{
	srand(GetTickCount());
	food.x = 10 + (rand() % 60 + 1) * 10;
	food.y = 10 + (rand() % 44 + 1) * 10;
	food.color = RGB(rand() % 256, rand() % 256, rand() % 256);
	food.r = rand() % 6 + 5;
	return;
}
int Eatself()
{
	int j;
	for (j = 1; j < snake.size; j++)
		if (snake.coor[j].x == snake.coor[0].x && snake.coor[j].y == snake.coor[0].y)
			return 1;
	return 0;
}
int Snakebump()
{
	int k = 10;
	if (snake.coor[0].x <= k || snake.coor[0].x >= lx - k || snake.coor[0].y <= k || snake.coor[0].y >= ly - k)
		return 1;
	return 0;
}
void Overmenu()
{
	cleardevice();
	IMAGE Game_over;//定义一个图片名.
	loadimage(&Game_over, "Picture\\game_over.png", lx, ly, 1);//从图片文件获取图像
	putimage(0, 0, &Game_over);//绘制图像到屏幕，图片左上角坐标为(0,0)
	while (1)
	{
		if (_getch() == '\x0d')//回车
			break;
	}
	cleardevice();
	//统计信息
	settextstyle(30, 0, "楷体");
	settextcolor(COLORREF RGB(0, 0, 0));
	outtextxy(220, 100, "GAME OVER");
	sprintf(s, "%d", mark);
	outtextxy(220, 150, "你的分数为");
	outtextxy(380, 150, s);
	outtextxy(220, 200, "按Enter重新游戏  ");
	outtextxy(220, 250, "按Esc退出游戏  ");
}
void Thank()
{
	IMAGE thank;//定义一个图片名.
	loadimage(&thank, "Picture\\06.png", lx, ly, 1);//从图片文件获取图像
	putimage(0, 0, &thank);//绘制图像到屏幕，图片左上角坐标为(0,0)
	Sleep(4000);
}
void Print()
{
	printf("\n");
	printf("\n");
	printf("感谢您的陪伴！\n");
	printf("您的分数是:   %d  分\n", mark);
	printf("\n");
	printf("游戏开发者:\n");
	printf("\n");
	printf("桃花涣小鱼干\n");
	printf("\n");
	getchar();
	getchar();
}
void Reset()
{
	int i;
	for (i = 0; i <= snake.size; i++)
		snake.coor[i].x = snake.coor[i].y = 0;
}
void Shift()
{
	keybd_event(0x10, 0, 0, 0);
	keybd_event(0x10, 0, KEYEVENTF_KEYUP, 0);
}
```

------

## 完整代码 (VS 2019)

```c
C
#include<math.h>
#include<time.h>
#include<conio.h>
#include<stdio.h>
#include<stdlib.h>
#include<process.h>
#include<windows.h>
#include<graphics.h>
#include <mmsystem.h>
#pragma comment(lib, "winmm.lib")

#define lx 640
#define ly 480
#define Sup 7//换肤模式突变点
#define MAX 500

int key,thread,flag, mark, p, ans, i = 1, threadend = 1;; char s[1000]; double speed = 80;

enum DIR//4个方向,enum为枚举类型
{
	PAUS,
	UP,
	DOWN,
	LEFT,
	RIGHT,
};
struct Food
{
	int x;
	int y;
	int r;//半径
	bool flag;
	DWORD color;
}food;
struct Snake
{
	int size;
	int speed;
	int dir;
	DWORD color1;
	DWORD color2;
	POINT coor[MAX];
}snake;


void BGM_ALL();										//全局BGM		
unsigned __stdcall BGM_GET(void* pArguments);		   //吃食物BGM		
void BGM_FAIL();									//失败BGM		
void Duce();										//引导菜单		
void Startmenu();									//开始菜单		
void Overmenu();									//结束菜单		
void Snakeinit();									//蛇初始化		
void Foodinit();		    						//食物初始化  		
void Init();										//初始化		
void Drawframe();									//画边框		
void Skinsystem(int i);								//皮肤系统		
void Drawsnake();									//普通切换皮肤模式		
void SupDrawsnake();								//快速切换皮肤模式		
void Drawfood();									//画食物		
void Draw();										//绘图		
void Firstcontrol();								//按键交互		
void Keycontrol();									//首次控制		
void Snakemove();									//蛇移动		
void Eatfood();										//吃食物
void Generatefood();								//生成食物
int Snakebump();									//蛇撞墙		
int Eatself();										//咬自己		
void Print();										//结束输出		
void Reset();										//重置		
void Thank();										//内测感谢名录		
void Shift();										//按下Shift键

void main()
{
	//多线程开始
	HANDLE   hThread;
	unsigned   threadID;
	hThread = (HANDLE)_beginthreadex(NULL, 0, &BGM_GET, NULL, 0, &threadID);
	BGM_ALL();
	Init();
	Startmenu();
	if (!key)//第一次开始按下shift键
	{
		Shift();
		key = 1;
	}
	while (1)
	{
		Sleep(speed);
		Draw();
		if (flag == 0)
		{
			Firstcontrol();
			flag = 1;
		}
		Snakemove();
		if (Eatself() || Snakebump())
		{
			mciSendString("close BGM1", 0, 0, 0);
			BGM_FAIL();
			break;
		}
		Eatfood();
	}
	Overmenu();
	switch (_getch())
	{
	case 13:
	{
		i = 1;
		speed = 80;
		flag = mark = ans = 0;
		Reset();
		threadend = 0;
		WaitForSingleObject(hThread, INFINITE);
		CloseHandle(hThread);
		threadend = 1;
		main();
	}
	break;
	case 27:
		Thank();
		closegraph();
		Print();
		break;
	}
	//多线程结束
	WaitForSingleObject(hThread, INFINITE);
	CloseHandle(hThread);
}

void BGM_ALL()
{
	mciSendString("open ./Music/Scheming_Weasel.mp3 alias BGM1", 0, 0, 0);
	mciSendString("play BGM1 repeat", 0, 0, 0);
}
unsigned __stdcall BGM_GET(void* pArguments)
{
	while (1)
	{
		if (thread)
		{
			mciSendString("open ./Music/Get.wav alias BGM2", 0, 0, 0);
			mciSendString("play BGM2 wait", 0, 0, 0);
			mciSendString("close BGM2 ", 0, 0, 0);
			thread = 0;
		}
		if (!threadend)
			break;
	}
	return 0;
}
void BGM_FAIL()
{
	mciSendString("open ./Music/Gameover.wav alias BGM3", 0, 0, 0);
	mciSendString("play BGM3 wait", 0, 0, 0);
	mciSendString("close BGM3 ", 0, 0, 0);
}
void Duce()
{
	if (p == 0) {
		p = 1;
		Sleep(300);
		IMAGE one;//定义一个图片名.
		loadimage(&one, "Picture\\01.png", lx, ly, 1);//从图片文件获取图像
		putimage(0, 0, &one);//绘制图像到屏幕，图片左上角坐标为(0,0)
		Sleep(1500);
		IMAGE two;//定义一个图片名.
		loadimage(&two, "Picture\\02.png", lx, ly, 1);//从图片文件获取图像
		putimage(0, 0, &two);//绘制图像到屏幕，图片左上角坐标为(0,0)
		Sleep(1500);
		IMAGE three;
		loadimage(&three, "Picture\\03.png", lx, ly, 1);//从图片文件获取图像
		putimage(0, 0, &three);//绘制图像到屏幕，图片左上角坐标为(0,0)
		Sleep(1400);
		IMAGE four;
		loadimage(&four, "Picture\\04.png", lx, ly, 1);//从图片文件获取图像
		putimage(0, 0, &four);//绘制图像到屏幕，图片左上角坐标为(0,0)
		Sleep(1500);
		IMAGE five;
		loadimage(&five, "Picture\\05.png", lx, ly, 1);//从图片文件获取图像
		putimage(0, 0, &five);//绘制图像到屏幕，图片左上角坐标为(0,0)
		Sleep(1400);
		cleardevice();
	}
}
void Startmenu()
{
	//Duce();
	while (_kbhit())
		getchar();
	IMAGE welcome;
	loadimage(&welcome, "Picture\\00.png", lx, ly, 1);//从图片文件获取图像
	putimage(0, 0, &welcome);//绘制图像到屏幕，图片左上角坐标为(0,0)
	while (1)
	{
		if (_getch() == 13)
			break;
	}
}
void Snakeinit()
{
	srand(GetTickCount());
	snake.color1 = RGB(0, 235, 229);
	snake.size = 4;
	snake.speed = 10;
	snake.dir = PAUS;
	int randomx = rand() % (lx - 100) + 50;
	int randomy = rand() % (ly - 100) + 50;
	for (int i = 0; i <= snake.size - 1; i++)
	{
		snake.coor[i].x = (randomx / 10) * 10 - 10 * i;	//（除10乘10类方格化处理）
		snake.coor[i].y = (randomy / 10) * 10;
	}
}
void Foodinit()
{
	int j;
	food.flag = 0;
	while (!food.flag)
	{
		Generatefood();
		for (j = 0; j < snake.size; j++)
			if (fabs(snake.coor[j].x - food.x) <= food.r && fabs(snake.coor[j].y - food.y) <= food.r)
				Generatefood();
		food.flag = true;
	}
}
void Init()
{
	initgraph(lx, ly);//初始化窗口，大小640*480,/*SHOWCONSOLE*/
	Snakeinit();//蛇初始化
	Foodinit();//食物初始化
}
void Drawframe()
{
	setlinestyle(PS_SOLID, 20);
	setlinecolor(RGB(93, 107, 153));
	rectangle(0, 0, 640, 480);
}
void Skinsystem(int q)
{
	int i;
	switch (q)
	{
	case 1://YES
		//蛇头
		setfillcolor(RGB(93, 107, 153));
		solidcircle(snake.coor[0].x, snake.coor[0].y, 6);
		//蛇身
		setfillcolor(RGB(150, 157, 177));
		for (i = 1; i <= snake.size - 1; i++)
			solidcircle(snake.coor[i].x, snake.coor[i].y, 5);
		break;
	case 2://YES
		//蛇头
		setfillcolor(RGB(186, 63, 110));
		solidcircle(snake.coor[0].x, snake.coor[0].y, 6);
		//蛇身
		setfillcolor(RGB(248, 237, 203));
		for (i = 1; i <= snake.size - 1; i++)
			solidcircle(snake.coor[i].x, snake.coor[i].y, 5);
		break;
	case 3://YES
		setfillcolor(RGB(127, 205, 238));
		solidcircle(snake.coor[0].x, snake.coor[0].y, 6);
		setfillcolor(RGB(244, 241, 236));
		for (i = 1; i <= snake.size - 1; i++)
			solidcircle(snake.coor[i].x, snake.coor[i].y, 5);
		break;
	case 4://YES
		//蛇头
		setfillcolor(RGB(221, 192, 179));
		solidcircle(snake.coor[0].x, snake.coor[0].y, 6);
		//蛇身
		setfillcolor(RGB(244, 241, 236));
		for (i = 1; i <= snake.size - 1; i++)
			solidcircle(snake.coor[i].x, snake.coor[i].y, 5);
		break;
	case 5://YES
		setfillcolor(RGB(113, 111, 114));
		solidcircle(snake.coor[0].x, snake.coor[0].y, 6);
		setfillcolor(RGB(244, 241, 236));
		for (i = 1; i <= snake.size - 1; i++)
			solidcircle(snake.coor[i].x, snake.coor[i].y, 5);
		break;
		//蛇头
	case 6://YES
		setfillcolor(RGB(206, 124, 128));
		solidcircle(snake.coor[0].x, snake.coor[0].y, 6);
		//蛇身
		setfillcolor(RGB(170, 175, 231));
		for (i = 1; i <= snake.size - 1; i++)
			solidcircle(snake.coor[i].x, snake.coor[i].y, 5);
		break;
	case 7://YES
		setfillcolor(RGB(223, 165, 161));
		solidcircle(snake.coor[0].x, snake.coor[0].y, 6);
		//蛇身
		setfillcolor(RGB(123, 130, 184));
		for (i = 1; i <= snake.size - 1; i++)
			solidcircle(snake.coor[i].x, snake.coor[i].y, 5);
		break;
	}
}
void SupDrawsnake()
{
	if (i <= 7)
	{
		Skinsystem(i);
		i++;
	}
	else
	{
		Skinsystem(1);
		i = 1;
	}
}
void Drawsnake()
{
	if (i <= 7)
		Skinsystem(i);
	else
	{
		Skinsystem(1);
		i = 1;
	}
}
void Drawfood()
{
	srand(GetTickCount());
	if (food.flag)
	{
		setfillcolor(RGB(rand() % 256, rand() % 256, rand() % 256));
		solidcircle(food.x, food.y, food.r);
	}
}
void Draw()
{
	cleardevice();
	BeginBatchDraw();//双缓冲绘图
	setbkcolor(RGB(204, 213, 240));//背景色
	cleardevice();//清空绘图设备
	Drawframe();//画边框
	//画蛇(两种模式)
	if (ans % Sup == 0 && ans >= 1)
		SupDrawsnake();
	else
		Drawsnake();
	Drawfood();//画食物
	EndBatchDraw();//双缓冲绘图结束
}
void Firstcontrol()
{
	switch (_getch())
	{
	case 'W':
	case'w':
	case 72:
		if (snake.dir != DOWN)
			snake.dir = UP;
		break;
	case 'S':
	case 's':
	case 80:
		if (snake.dir != UP)
			snake.dir = DOWN;
		break;
	case 'A':
	case 'a':
	case 75:
		if (snake.dir != RIGHT)
			snake.dir = LEFT;
		break;
	case 'D':
	case 'd':
	case 77:
		if (snake.dir != LEFT)
			snake.dir = RIGHT;
		break;
	case 13:
		Firstcontrol();
		break;
	}
}
void Keycontrol()//按键控制
{
	if (_kbhit())
	{

		char key = _getch();
		//72,80,75,77
		switch (key)
		{
		case 'W'://上
		case'w':
		case 72:
			if (snake.dir != DOWN)
				snake.dir = UP;
			break;
		case 'S'://下
		case 's':
		case 80:
			if (snake.dir != UP)
				snake.dir = DOWN;
			break;
		case 'A'://左
		case 'a':
		case 75:
			if (snake.dir != RIGHT)
				snake.dir = LEFT;
			break;
		case 'D'://右
		case 'd':
		case 77:
			if (snake.dir != LEFT)
				snake.dir = RIGHT;
			break;
		case 13:
			getchar();
			break;
		}
	}
}
void Snakemove()
{
	int i;
	for (i = snake.size - 1; i > 0; i--)
		snake.coor[i] = snake.coor[i - 1];
	Keycontrol();
	switch (snake.dir)
	{
	case UP:
		snake.coor[0].y -= snake.speed;
		break;
	case DOWN:
		snake.coor[0].y += snake.speed;
		break;
	case LEFT:
		snake.coor[0].x -= snake.speed;
		break;
	case RIGHT:
		snake.coor[0].x += snake.speed;
		break;
	case PAUS:
		getchar();
		break;
	}
}
void Eatfood()
{
	if (food.flag && fabs(snake.coor[0].x - food.x) <= food.r && fabs(snake.coor[0].y - food.y) <= food.r)
	{
		thread = 1;//多线程开始执行
		i++;
		ans++;
		snake.color1 = food.color;
		if (speed > 60)
			speed -= 0.05;
		food.flag = false;
		mark += food.r*2/3;
		snake.size += food.r*2/3;
		//显示分数
		settextstyle(25, 0, "微软雅黑");
		settextcolor(COLORREF RGB(0, 0, 0));
		sprintf_s(s, "%d", mark);
		outtextxy(food.x, food.y, s);
	}
	int j;
	while (!food.flag)
	{
		Generatefood();
		for (j = 0; j < snake.size; j++)
			if (fabs(snake.coor[j].x - food.x) <= food.r && fabs(snake.coor[j].y - food.y) <= food.r)
				Generatefood();
		food.flag = true;
	}
}
void Generatefood()
{
	srand(GetTickCount());
	food.x = 10 + (rand() % 60 + 1) * 10;
	food.y = 10 + (rand() % 44 + 1) * 10;
	food.color = RGB(rand() % 256, rand() % 256, rand() % 256);
	food.r = rand() % 6 + 5;
	return;
}
int Eatself()
{
	int j;
	for (j = 1; j < snake.size; j++)
		if (snake.coor[j].x == snake.coor[0].x && snake.coor[j].y == snake.coor[0].y)
			return 1;
	return 0;
}
int Snakebump()
{
	int k = 10;
	if (snake.coor[0].x <= k || snake.coor[0].x >= lx - k || snake.coor[0].y <= k || snake.coor[0].y >= ly - k)
		return 1;
	return 0;
}
void Overmenu()
{
	cleardevice();
	IMAGE Game_over;//定义一个图片名.
	loadimage(&Game_over, "Picture\\game_over.png", lx, ly, 1);//从图片文件获取图像
	putimage(0, 0, &Game_over);//绘制图像到屏幕，图片左上角坐标为(0,0)
	while (1)
	{
		if (_getch() == '\x0d')//回车
			break;
	}
	cleardevice();
	//统计信息
	settextstyle(30, 0, "楷体");
	settextcolor(COLORREF RGB(0, 0, 0));
	outtextxy(220, 100, "GAME OVER");
	sprintf_s(s, "%d", mark);
	outtextxy(220, 150, "你的分数为");
	outtextxy(380, 150, s);
	outtextxy(220, 200, "按Enter重新游戏  ");
	outtextxy(220, 250, "按Esc退出游戏  ");
}
void Thank()
{
	IMAGE thank;//定义一个图片名.
	loadimage(&thank, "Picture\\06.png", lx, ly, 1);//从图片文件获取图像
	putimage(0, 0, &thank);//绘制图像到屏幕，图片左上角坐标为(0,0)
	Sleep(4000);
}
void Print()
{
	printf("\n");
	printf("\n");
	printf("感谢您的陪伴！\n");
	printf("您的分数是:   %d  分\n", mark);
	printf("\n");
	printf("游戏开发者:\n");
	printf("\n");
	printf("桃花涣小鱼干\n");
	printf("\n");
	getchar();
	getchar();
}
void Reset()
{
	int i;
	for (i = 0; i <= snake.size; i++)
		snake.coor[i].x = snake.coor[i].y = 0;
}
void Shift()
{
	keybd_event(0x10, 0, 0, 0);
	keybd_event(0x10, 0, KEYEVENTF_KEYUP, 0);
}
```

------

## `感谢名录`

<font size=5 color=#D1DAF2>**Do_r**</font>

<font size=5 color=#0E0E0E>**Jake**</font>

<font size=5 color=#007CFF>**Mine**</font>

<font size=5 color=#9D87D2>**Paraboy**</font>

<font size=5 color=#8FD1EC>**Tremor.**</font>

<font size=5 color=#B884F2>**Yeyu**</font>

<font size=5 color=#8FD1EC>**沫殇心**</font>

## `感谢陪伴！`

# `END`
