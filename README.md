# ANIMAL
这是一个实验项目
#include <Rtc_Pcf8563.h>
#include <Adafruit_NeoPixel.h>//include声明函数
Rtc_Pcf8563 rtc;
#include "I2Cdev.h"
#include "Wire.h"
#include <SHT2x.h>
#include "U8glib.h"
#define PIXEL_PIN       A0    
//-------字体设置，大、中、小
#define setFont_L u8g.setFont(u8g_font_7x13)
#define setFont_M u8g.setFont(u8g_font_fixed_v0r)
#define setFont_S u8g.setFont(u8g_font_fixed_v0r)
#include <SHT2x.h>
#define setFont_SS u8g.setFont(u8g_font_fub25n)
/*
font:
 u8g_font_7x13
 u8g_font_fixed_v0r
 u8g_font_chikitar
 u8g_font_osb21
 u8g_font_courB14r
 u8g_font_courB24n
 u8g_font_9x18Br
 */
//屏幕类型--------
U8GLIB_SSD1306_128X64 u8g(U8G_I2C_OPT_NONE);

#define init_draw 10	//主界面刷新时间
float sensor_tem;

unsigned long timer_draw,timer;
int pkj=0;
String dateStr, ret;
void temper(int s){
   u8g.setFont(u8g_font_5x8);
   u8g.setPrintPos(pkj, 64); 
  u8g.print(s);
}
void read()
{
  sensor_tem = SHT2x.GetTemperature() ;//把获得的温度值赋给变量sensor_tem
  Serial.println(sensor_tem);//将数据从Arduino传递到PC且单独占据一行，此数据可在串口监视器中看到
  Serial.print("--");   
  Serial.println(SHT2x.GetHumidity());
  delay(100);

}
void setup() {

  Serial.begin(9600);
  analogReference(INTERNAL);  //调用板载1.1V基准源
 Wire.begin(); 

  setRtcTime(17, 5, 18, 1, 00, 00, 00);
  timer=millis();
}

void loop() {

    read();
  ret = getRtcTimeString();
  if (millis() - timer_draw > init_draw)
  {
    pkj-=4;
    u8g.firstPage();
    do {
      draw();
    }
    while ( u8g.nextPage() );
    timer_draw = millis();
  }
}

//主界面，可自由定义
void draw()
{
  setFont_L;
  u8g.setPrintPos(4, 16);
  u8g.print(rtc.formatDate(RTCC_DATE_US));
  u8g.print("    ");
  switch (rtc.getWeekday()) {
    case 1:
      u8g.print("Mon");
      break;
    case 2:
      u8g.print("Tue");
      break;
    case 3:
      u8g.print("Wed");
      break;
    case 4:
      u8g.print("Thu");
      break;
    case 5:
      u8g.print("Fri");
      break;
    case 6:
      u8g.print("Sat");
      break;
    case 7:
      u8g.print("Sun");
      break;
  }
 temper(sensor_tem);
  setFont_SS;
  u8g.setPrintPos(18, 49);
  u8g.print(rtc.getHour());
  u8g.setPrintPos(55, 46);
  if (rtc.getSecond() % 2 == 0)
    u8g.print(":");
  else
    u8g.print(" ");
  u8g.setPrintPos(68, 48);
  if (rtc.getMinute() < 10)
  {
    u8g.print("0");
    u8g.print(rtc.getMinute());
  }
  else
    u8g.print(rtc.getMinute());
}

void setRtcTime (byte _year, byte _month, byte _day, byte _week, byte _hour, byte _minute, byte _sec)
{
  //clear out all the registers
  rtc.initClock();
  rtc.setDate(_day, _week, _month, 0, _year);
  rtc.setTime(_hour, _minute, _sec);
}

String getRtcTimeString() {
  dateStr = rtc.formatDate(RTCC_DATE_US);
  dateStr += " ";
  dateStr += rtc.formatTime(RTCC_TIME_HMS);

  return dateStr;
}
