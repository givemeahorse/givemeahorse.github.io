# 个人游戏开发实现作品集
# 目录
- [1.游戏简介](#游戏简介)
- [2.策划思路](#策划思路)
- [3.技术路线](#技术路线)
  - [3.1技术总图](#技术总图)
  - [3.2模型处理](#模型处理)
# 游戏简介
<div align=center>
<img src=img/figure_steam.png>
</div>  
爪印（PawPrint）是写给那些现实生活中失去自己爱猫人群的治愈经营游戏。无论猫咪是自然离世、生病去世、亦或是无声无息的离开，喵星人带给人的温暖永远不会消失。希望通过这个works可以带给人们慰藉，世界破破烂烂，小猫缝缝补补。</br>
PawPrint is a healing game for people who have lost their cats in real life. Whether your cat dies naturally, gets sick, or leaves without a word, the warmth that cats bring to you will never die. I hope this works can bring people comfort, the world is in tatters, and kittens are mending.

# 策划思路
策划路线：以休闲运营为主调，设计家园场景和喵星球场景。在家园场景中实现喂养清洁陪伴、DIY自定义猫形象、日记记录等功能，喵星球场景中实现自由探索功能。以手游为目标，后期移植pc。将受众人群的心理期待为主线，尽可能多的体现人与猫之间的纽带，弱化人的控制，尽可能多的还原猫咪日常状态。  
Planning route: Focus on leisure operation, design home scene and Meow planet scene. In the home scene, the functions of feeding and cleaning, DIY custom cat image, diary recording and so on are realized, and the function of free exploration is realized in the cat planet scene. Target mobile games as the target, later transplant pc. The goal is to take the psychological expectations of the audience as the target, to reflect the bond between people and cats as much as possible, weaken the control of people, and restore the daily state of cats as much as possible.  
![image](img/homescene.jpg)
![image](img/planetscene.jpg)
![image](img/graffi.jpg)
![image](img/shop1.png)

# 技术路线
## 技术总图
![image](img/process3.png)
## 模型处理
猫角色的模型的的创建主要考虑了三个因素，一是猫咪角色的形象还原，二是摈弃之前游戏创作时一味想亲身体验从建模到骨骼绑定、刷权重、做动画导致的体力时间浪费，三是兼顾性能优化和配合shader毛发片元挤出的质量观感的中低模，决定从网上直接找mesh和骨骼动画合格的产品。  
原模型在Avator动画传递时发现后腿部变形严重，骨骼绑定需要修改。（还好不算多，否则直接abandon了）

制作猫咪模型胖瘦变化，对比了几种方法，最后决定用blendshape在3dmax里调整好mesh角点的缩放极值，导出参数在unity通过调值来动态控制猫咪胖瘦。这一技术多用于表情变化与捏脸中，在《爪印》中刚好能契合简单的形体变化需求。


