---
layout:     post
title:      Dynamics CRM中Date and Time类型字段各种不同behavior的区别
subtitle:   
date:       2021-01-19
author:     Stone
header-img: img/post-bg-ios9-web.jpg
catalog: true
tags:
    - Dynamics CRM
	- SSRS
---

## Dynamics CRM中Date and Time类型字段各种不同behavior的区别
字段类型Date and Time有三种不同给的behavior: **User Local**, **Date Only**和**Time-Zone Independent**。
下面我们就来说一下他们之间的区别。我们通过在一个实体中分别创建集中不同的Behavior来比较他们的区别：
1. User Local -- Format为Date Only
![User Local-Date Only](https://github.com/hdjgreen/dynamicscrm.github.io/blob/raw/master/img/20210119/1.png)
2. User Local -- Format为Date and Time
![User Local -- Format为Date and Time](:/91b9a2ebb4484ab8a9f9a9dfd655c1a0)
3. Date Only(它的格式只有Date only一种)
![Date Only(它的格式只有Date only一种)](:/c90293608dc148d68bdf3a5b6d4ea03b)
4. Time-Zone Independent -- Format为Date Only
![Time-Zone Independent -- Format为Date Only](:/1e2eb55b754c4a1fa97081c81899e42b)
5. Time-Zone Independent -- Format为Date and Time
![Time-Zone Independent -- Format为Date and Time](:/3c70c2a4f75d438fa45435a86a675fb4)

我们创建一条记录并将他们的值设置为如下:
![b7a905bff9d2a7b65042453197ca14f0.png](:/4f5059ccd44947cbaba61f477a8d7f7c)
保存后，我们去数据库看下他们他们在数据库的值是什么：
![740c49d269094b17639cba26768d47ff.png](:/2ea919056e5049abad3df39c2eef215c)

这里需要说明一下，我们在CRM设置系统的时区设置是UTC + 8
![51eb9dd3c34eb1ab5d61e5432e03bb12.png](:/29ed1de3f1fa4f5b9145f1c6d5892b93)

通过比较我们能够发现他们的区别：
1. **User Local -- Format为Date Only**
前端只会显示日期不会显示时间，但是会把这个值默认为UTC + 8的时区，时间为1/19/2021 00:00:00.000，但是在数据库中会存储为UTC + 0的时间，也就是需要减掉8个小时，所以是2021-01-18 16:00:00.000，也就是数据库截图中的第三列
2. **User Local -- Format为Date and Time**
我们把第一个User Local格式为Date Only的如果理解了，就不难明白这个为什么在数据库中是2021-01-19 00:00:00.000，因为我们在前端界面设置的时间为1/19/2021 8:00 AM，系统默认为UTC + 8的时区，那么存储在数据库中的时间为UTC + 0，也就是减掉8个小时为2021-01-19 00:00:00.000，也就是数据库截图中的第四列
3. **Date Only(它的格式只有Date only一种)**
这种Date Only和Time-Zone Independent都是不区分时区的，也就是和时区无关的，可以理解为在前端填的是什么，数据库中记录的就是什么，无论你是什么时区。因为这个是Date Only的，所以前端不会显示时间，那么数据库就会记为2021-01-19 00:00:00.000，也就是数据库截图中的第一列
4. **Time-Zone Independent -- Format为Date Only**
同第三种一样，不区分时区，只显示日期，不显示时间，数据库记录为2021-01-19 00:00:00.000，也就是数据库截图中的第五列，通过尝试可以看出Behavior如果为Date Only和Time-Zone Independent的Date Only，那么它们在数据库的表现形式是一样的，并没有区别，在前端页面的显示也是相同的。
5. **Time-Zone Independent -- Format为Date and Time**
同样与时区无关，我们前端设置为1/19/2021 8:00 AM，那么数据库中就记为2021-01-19 08:00:00.000，也就是数据库截图中的第二列

***如果用户不是多个时区的，这一块的差别是可以忽略的，很少有人会去关注他们在数据库中存储的值是什么，但是如果你需要写SSRS报表，需要直接从数据库中取数据，那么数据库和前端的差别就非常关键了，你需要知道Date and Time字段它的Behavior是什么类型，然后在报表中来决定是否需要换算，这些正是我最近在开发报表中所遇到的，所以大家如果有和我一样的情况，也请注意！***
<br>
另外还在说一下这三种Behavior的一个区别就是，一旦建了一个Date and Time字段，behavior是Date Only或者Time-Zone Independent的，那么这个字段在保存后，它的Behavior就不能更改了；
但如果behavior是User Local的，那么即使保存后，也是可以更改的。如下图：
- User Local
![fcea92e96749591488c38a6415cd3977.png](:/79009aac3d954b558b606cf07a2423d8) 
<br>
- Date Only
![c81f6b6d3e62988bc5288fc80119c3a3.png](:/3032f7ac3a524a1fa5154ee9f5575137)
<br>
- Time-Zone Independent
![b9348faa8db2653b671aa3912db97950.png](:/f20bd067f1d848a18fa46618bd00fac4)
