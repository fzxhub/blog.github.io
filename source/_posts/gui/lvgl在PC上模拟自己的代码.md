---
title: lvgl在PC上模拟自己的代码
date: 2021-09-28
author: fzxhub
cover: true
img: 
summary: LGVL是一个开源的嵌入式GUI，在PC上使用Vscode模拟自己的代码。
categories: gui
tags:
  - lgvl
  - gui
  - 嵌入式
  - vscode
---

## 背景
- 使用官方VScode模拟器项目
- 使用lvgl8.0代码
- 使用macos系统
- 也就是我仓库lvgl_sim_vscode前提之下

## 建立自己的代码文件
1. 在lv_examples/src下新建文件夹lv_myapp
2. lv_examples/src/lv_myapp新建lv_myapp.c和lv_myapp.h

### lv_myapp.h中代码如下

``` c
/**
 * @file lv_myapp.h
 *
 */

#ifndef LV_MYAPP_H
#define LV_MYAPP_H

#ifdef __cplusplus
extern "C" {
#endif

/*********************
 *      INCLUDES
 *********************/

/*********************
 *      DEFINES
 *********************/

/**********************
 *      TYPEDEFS
 **********************/

/**********************
 * GLOBAL PROTOTYPES
 **********************/
void lv_myapp(void);

/**********************
 *      MACROS
 **********************/

#ifdef __cplusplus
} /* extern "C" */
#endif

#endif /*LV_MYAPP_H*/
```

### lv_myapp.c中代码如下

``` c
#include "../../lv_demo.h"

void lv_myapp(void)
{
    lv_obj_t *scr = lv_scr_act();
    lv_obj_t *label1 = lv_label_create(scr);
    lv_label_set_text(label1,"I am fzxhub!");
    lv_obj_align(label1,LV_ALIGN_CENTER,0,0);

}
```

## 引用自己头文件

在lv_examples/lv_demo.h中添加自己的头文件

``` c
#include "src/lv_demo_widgets/lv_demo_widgets.h"
#include "src/lv_demo_benchmark/lv_demo_benchmark.h"
#include "src/lv_demo_stress/lv_demo_stress.h"
#include "src/lv_demo_keypad_encoder/lv_demo_keypad_encoder.h"
#include "src/lv_demo_music/lv_demo_music.h"
#include "src/lv_myapp/lv_myapp.h"
```

## 在main.c中调用自己的代码

在hal_init();之后调用自己的函数即可，然后运行可查看效果。

``` c
  /*Initialize LVGL*/
  lv_init();

  /*Initialize the HAL (display, input devices, tick) for LVGL*/
  hal_init();

//  lv_example_switch_1();
//  lv_example_calendar_1();
//  lv_example_btnmatrix_2();
//  lv_example_checkbox_1();
//  lv_example_colorwheel_1();
//  lv_example_chart_6();
//  lv_example_table_2();
//  lv_example_scroll_2();
//  lv_example_textarea_1();
//  lv_example_msgbox_1();
//  lv_example_dropdown_2();
//  lv_example_btn_1();
//  lv_example_scroll_1();
//  lv_example_tabview_1();
//  lv_example_tabview_1();
//  lv_example_flex_3();
//  lv_example_label_1();

//  lv_demo_widgets();
//  lv_demo_keypad_encoder();
//  lv_demo_benchmark();
//  lv_demo_stress();
//  lv_demo_music();
    lv_myapp();
```