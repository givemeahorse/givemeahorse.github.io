# 个人游戏开发实现作品集
# 目录
- [1.游戏简介](#游戏简介)
- [2.策划思路](#策划思路)
- [3.技术路线](#技术路线)
  - [3.1技术总图](#技术总图)
  - [3.2模型处理](#模型处理)
  - [3.3毛发效果](#毛发效果)
  - [3.4形象自定义](#形象自定义)
  - [3.5状态切换](#状态切换)
  - [3.6日记记录](#日记记录)
  - [3.7商品交易](#商品交易)
  - [3.8自由探险](#自由探险)
  - [3.9模块功能](#模块功能)
- [4.游戏发表](#游戏发表)
- [5.其他游戏](#其他游戏)
- [6.写在最后](#写在最后)
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
猫角色的模型的的创建主要考虑了三个因素，一是猫咪角色的形象还原，二是摈弃之前游戏创作时一味想亲身体验从建模到骨骼绑定、刷权重、做动画导致的体力时间浪费，三是能兼顾性能优化和配合shader毛发片元挤出的质量观感的中低模，决定从网上直接找mesh和骨骼动画合格的产品。  
原模型在Avator动画传递时发现后腿部变形严重，骨骼绑定需要修改（还好不算多，否则直接abandon了）。
![image](img/bone.png)
制作猫咪模型胖瘦变化，对比了权重贴图、blendshape等几种方法，最后决定用blendshape在3dmax里调整好mesh角点的缩放极值，导出参数在unity通过调值来动态控制猫咪胖瘦。这一技术多用于表情变化与捏脸中，在《爪印》中刚好能契合简单的形体变化需求，每次改变后再bake一次就行，因为是中低模，此处的性能消耗可以直接忽略。
![image](img/blendshape1.png)
![image](img/blendshape2.png)  
该模型材质默认整体统一，但为了后期配合毛发shader在不同部位的生长情况不同（眼睛、爪垫、耳内、鼻子等位置是不长毛的，且脸部、头顶、躯干、四肢毛发稀疏、长度也不同），所以需要将整体材质进行拆分。这里用多维子材质功能在3dmax中提前将不同部位材质区进行分割，方便后期多个shader对应各自部位进行分开调整来达到我想要的效果。
![image](img/submaterials.png)  
![image](img/submodel.png)  
## 毛发效果
在想要做出毛发前，查阅了大量文献和资料。最初是想做出《动物派对》的毛发自然效果，但是考虑了该游戏发布将手机端放在首位，所以借鉴了《王者荣耀》中妲己的尾巴的毛发效果。该shader是在urp下成型的通过片元挤出,多pass层间断生成来达到毛发的散射加透光的效果。原理图如下：  
![image](img/fur1.png)  
这个shader我研究了比较久，在确定使用这个shader之前，我自己有写一个有权重贴图的shader想达到效果，但是在从vert传递涂鸦操作时的精准纹理坐标到frag总是报错，一是geom中不能进行纹理采样，二是像素着色器中二次修改顶点坐标是不可能的。所以最后直接采用了妲己毛发shader进行部分修改：
手指涂抹猫的mesh的区域进行涂色的信息进行记录，之后在将这些区域和颜色对shader的baseTexture重新SetPixels，这样能绕开在shader中坐标过渡不精准的output._FurWeight = SAMPLE_TEXTURE2D(_FurWeightMap, sampler_FurWeightMap, uvWeight).r类似问题。  
最后思路：调用基础贴图的像素进行渲染，涂鸦的功能我通过外部的baseTexture指定这个shader使用。这样整体颜色、涂鸦颜色的问题可以一起解决。  
此外毛发长度、毛发疏密、毛发层数直接用参数调用，配合前期的多维子材质分块进行动态调整，就能达到想要的效果了。  
但是毛发层数是个很耗性能的drawcall，在调整毛发效果时尽量调节间距而不是单纯增加层数，来达到长毛的效果。最终效果和参数代码如下：  
![image](img/effect.png)  
![image](img/effect1.png)  
![image](img/furshader2.png) 
![image](img/shaderPar.png) 
![image](img/furshader3.png) 
初始化毛发效果做好后，也同时留好了调整毛发长度、整体毛色预设和自定义、涂鸦颜色的接口了，之后UGUI控件和涂抹来调整的空间就留给玩家了。
![image](img/graffi1.png) 

## 形象自定义
猫咪胖瘦、毛发长度两项参数玩家通过slider控制，其中猫咪胖瘦调整后会重新bake确保形体坐标一致。整体毛发颜色可采用预设的橘猫、狸花猫、奶牛猫等几种贴图一键设置，也可以独立在colorPanel选择颜色进行整体毛发颜色更换。涂鸦为选择颜色后在猫身上对应区域进行涂抹。这些操作会进行actionRecord的记录，来帮助玩家撤销、清除或保存自己的操作。  
保存毛发信息的方法因为像素数组容量大每次保存会轻微卡顿，故采用异步保存的方式，将主程中的毛发颜色参数单独提出去在外部进行序列化，可节省计算时间。  
在处理用户涂抹时指头离开的方法中遇到无法在确定触摸为true中监控变为flase的事件，采用协程的方式来处理。协程在该游戏中有大量使用，除了DOTween外，在处理时间线需要判定时，协程能较方便的进行判定和触发。  
## 状态切换
游戏在家园场景中设计了猫咪可以自由在屋子里进行随机行走、奔跑、伸懒腰、舔爪、休息、吃饭、喝水等移动和动画，因为我此处想还原猫咪的自由状态，弱化玩家的控制，来轻松的带给玩家猫咪还陪伴在身边的真实感。  
首先给家园场景中生成导航代理的区域作为猫咪的活动空间，为了减少穿模的视觉劣质感，需将Agent区域的半径设置和猫的boundingbox大致一致。之后猫的移动依靠在代理区域内随机生成坐标位置驱动Agent的SetDestination来代理猫咪进行移动。  
动画部分采用Animator来链接不同动画触发与结束，形成移动动画和原地动画，例如行走、奔跑到房间内某处，躺下伸懒腰、坐下发呆、饥饿口渴值为零时加粮加水会奔跑到饭碗处进食喝水等状态切换。此处用方法调用移动代理与Animator来实现随机的猫咪状态切换。
![image](img/ani.png) 
![image](img/a2.png) 
![image](img/a3.png) 
![image](img/a4.png) 
![image](img/a5.png) 
![image](img/randomaction.png) 
![image](img/randomaction1.png) 
## 日记记录
考虑到最初设计的受众玩家的情感互动需求，我想加入一些能够寄托感情的功能。最初规划了几个功能：上传和猫咪的合照、写日记、访问别人的家园和公开日志、调用chatGPT的API预训练一个模型代替去世的猫咪角色和玩家进行对话等，能够完善治愈系这个主题。无奈做到这里的时候我的本职工作太忙了，规划时间不够，只能选取日记记录这个最简单的功能来做。一个个人游戏开发者的无奈吧。  
在日记记录这个功能中，玩家的思念、遗憾等都可以在日记本面板中写下来，日记内容通过序列化和反序列化保存和读取，供玩家随时查看。
![image](img/diary.png) 
## 商品交易
为了模拟日常生活中给猫咪购物喂食添水场景需求，做了这个商品交易功能。点击商店或房间内的箱子模型，可以进入商店与储藏界面。在这里，玩家可以通过点击对应的猫粮、罐头、猫条等商品进行购买，购买后的商品存储在个人仓库中，点击对应的商品进行投喂。同时也可以点击猫砂盆触发清洁动画进行铲屎清洁，这些功能同时和猫的饥饿、口渴、清洁、心情状态值进行绑定，来完成玩家保持爱猫最好状态的动态需求。其中金币为是在另一个喵星球场景中进行赚取，也是为了促进玩家在两个场景中多切换进行体验。  
这部分中功能实现和上条大同小异，多为数据传递与存储。大部分时间都花在UI的Panel、Scroll View和Viewport的处理上了。
![image](img/shop1.png) 
## 自由探险
这个游戏设计了喵星这个场景，能够让玩家操控猫咪在喵星球这个世界中自由探索，保持人和爱猫的牵绊联系。我想搭建一个塞尔达风格的温暖场景，出于时间的考虑，果断放弃自己进行地编的过程。筛选了大量场景模型与光照效果，最后采用如图的样式场景，同时加入了几个喵星守护神的角色模型放置在地图的不同位置。  
自由探险模块主要涉及了地形碰撞体、猫咪CharacterController，动画切换、摇杆控制、金币赚取和场景单例模式数据传递。
![image](img/p3.png) 
![image](img/p1.jpg) 
![image](img/p2.jpg) 
![image](img/p4.png) 
![image](img/p5.png) 
![image](img/p6.png) 
## 模块功能
支撑游戏的其他小模块包含ColorPicker、DOTween、niceTouch等外部resource和一些EventTrigger、mesh计算、空间数学换算、UGUI的功能使用、数据读取保存、触摸命令等。
# 游戏发表
《爪印》设计之初是为了做手游版，在游戏完成后选择TapTap平台发布，在历经了多次审核中遇到的问题和debug后，最终成功登录。手游版发表成功后，进行了PC移植，之后在Steam发布。  
目前申请软著在审核中。  
发表这个游戏一是主要想学习体验下游戏落地的全过程，二是游戏想法的最终表达。发表前后的环节中，遇到了设备过关、版本过审、产品数据保密、宣发成本、流量数据监控等一些列现实问题。有些问题，可以适应与解决，有些问题，个人无法主宰，但是游戏发表这一环节中学到的经验也算是一种treasure吧，至少能帮我的下一款游戏发表时少走一些弯路。  
![image](img/tap.jpg) 
![image](img/steamstore.png) 

# 其他游戏
个人正在验证另一款游戏，名字叫做《TA不是小猫咪》，核心玩法是真实物理环境反馈的猫咪对战游戏，定位为单击剧情游戏，以猫为主人公视角讲述寻找丢失的朋友，揭露黑暗真相的故事。剧情受《精灵与萤火意志》影响，核心玩法受《动物派对》与《无极道人》启发。目前已完成建模、骨骼与蒙皮权重、K帧动画、材质赋予、毛发生成等模型基础工作，正在完成地编、骨骼碰撞体与模型关节力学测试以及物理驱动动画的验证。
![image](img/image3.png) 
![image](img/image1.png) 
![image](img/box.png) 
![image](img/image.png) 
![image](img/furkitty.png) 
![image](img/naoyang.png) 

# 写在最后
《爪印》的开发是利用闲暇之余进行的，从冒出这个idea到策划再到实施至最后落地发表，是唯一一个走完全程的游戏。之前还有两个产品，第一个因为亲历建模、骨骼动画的时间过长和难以还原主动ragdoll的力学交互而陷入长久的验证测试，第二个因为产品定位体量太大，确实对于个人开发者来说很难在有限的时间内完成而放弃。吸取了前两个游戏的经验教训，在《爪印》开发时，尽量做一个人力所能及的事，眼光往当前看。尽管如此，开发的时间仍然超出我的计划。前期设计了很多温暖有意思的功能环节，但是受限于主业工作的忙碌，不得不取消一些计划。坦白来讲，这款游戏是有烂尾的成分的，我自己深知这一点，但值得庆幸的是，在零零碎碎、褒贬不一的评论中，我至少能从负面评价中找到我同样遗憾的痛点，至少证明我那些没有落地的想法是被在意的。  
个人开发的过程是充实且忐忑的，一方面，能看到我的灵感和想法能被自己的产品化出来，另一方面，过程中遇到的问题让我时常觉得是在闭门造车。策划游戏时文思泉涌，真正实施起来是要付出想法太飘逸的代价的，最终会实现，但是时间与精力成本无法预估。但从另一个角度考虑，理想主义和现实的纠葛本身也是游戏的灵魂。  
写这个游戏使用了AI产品来辅助，例如chatGPT帮助代码难点的实现、SD，midjourney帮助UI图标和app图标的生成。这让我对于未来游戏开发有了一些思考，例如：原画、定型、建模、材质、骨骼绑定、动画的过程是个体力与耐心堆砌的过程，这也是我有一些角色灵感素描出来的时候却不敢真正进一步实施的原因，那么从原画到制作模型、刷权重、骨骼绑定、K帧动画是否可以借助AI进行完成，这至少对于个人开发者来说是个乐观的畅想。例如Wonder Studio、Genie、Tripo的智能生产3d模型的过程，以及类似于sora、Pika、Stable Video Diffusion的图片生成视频对于游戏CG、转场、宣发的促进作用，甚至对于下沉市场的短平快的低端游戏的一体成型以及游戏企业训练自己的AI模型来做标件。自然语言到解决方案、图片、视频、模型、地编、剧情策划、战斗设计等的成型能够更快，那么制作游戏的成本将会下降很多。科技爆发的时代，保持OPEN吧。  
接下来有三个游戏，亮点核心玩法已经策划完成，但是鉴于之前的经验，打算仅做demo了。individual developer的成就感是一时绽放的高潮之花，只有保持不断学习与创新的态度，珍惜迸发的灵感，同时善于总结经验教训，付诸于实践，坚持游戏创作的热爱，才不会凋谢。  

The development of Paw Prints was carried out in spare time, from the emergence of the idea to the planning and implementation to the final landing and publication, and is the only game to go through the whole process. Before that, there were two products, the first one was abandoned because the hands-on modeling, bone animation time is too long, and it is difficult to restore the mechanical interaction of active ragdoll, and the second one was abandoned because the product positioning volume is too large, and it is indeed difficult for individual developers to complete in a limited time. Drawing on the lessons of the first two games, in the development of Paw Prints, try to do what one can do, and look at the present. Still, development took longer than I had planned. In the early stage, I designed many warm and interesting functional links, but due to the busy work of the main business, I had to cancel some plans. To be honest, the game had its share of bad endings, and I knew it, but thankfully, in the scattered, mixed reviews, I was able to find at least as much pain in the negative reviews as I did in proving that my ideas that didn't land were taken seriously.  
The process of personal development is full and disturbing. On the one hand, I can see that my inspiration and ideas can be translated into my own products. On the other hand, the problems encountered in the process often make me feel that I am working behind closed doors. When planning the game, the writing springs up, and the real implementation is to pay the price of the idea is too elegant, and it will eventually be realized, but the cost of time and effort can not be predicted. But from another perspective, the entanglement between idealism and reality is itself the soul of the game.  
Writing this game uses AI products to assist, such as chatGPT to help the realization of code difficulties, SD, midjourney to help the generation of UI ICONS and app ICONS. This gives me some ideas for the future of game development, such as: The process of original painting, stereotyping, modeling, material, bone binding, and animation is a process of physical strength and patience, which is also the reason why I dare not really further implement when I have some character inspiration sketches, so can I complete the process from original painting to making models, brush weights, bone binding, and K-frame animation with the help of AI? That's an optimistic scenario, at least for individual developers. For example, Wonder Studio, Genie, Tripo's intelligent 3d model production process, and similar to sora, Pika, Stable Video Diffusion image generation video for the game CG, transition, promotion. Even for the integration of short and fast low-end games in the sinking market, and game companies train their own AI models to make standard parts. Natural language to solutions, images, videos, models, maps, story planning, battle design, etc. can be made faster, and the cost of making a game will drop a lot. In the age of technology explosion, keep it OPEN.  
In the next three games, the core gameplay has been planned and completed, but in view of the previous experience, I intend to do only demo. The sense of achievement of individual developer is the climax flower that blooms for a while. Only by maintaining the attitude of continuous learning and innovation, cherishing the burst of inspiration, at the same time, being good at summing up experience and lessons, putting them into practice, and persisting in the love of game creation, will it not fade away.
