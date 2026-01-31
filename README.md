<!DOCTYPE html>
<html lang="zh-CN"> 
<head> 
    <meta charset="UTF-8"> 
    <meta name="viewport" content="width=device-width, initial-scale=1.0"> 
    <title>泡泡末世：连贯叙事版</title> 
    <script src="https://cdn.tailwindcss.com"></script> 
    <style> 
        @import url('https://fonts.googleapis.com/css2?family=Noto+Serif+SC:wght@200;300;400;700&display=swap'); 

        :root { 
            --bloom-color: rgba(255, 255, 255, 0.45);
            --tech-blue: rgba(147, 197, 253, 0.8);
            --text-glow: 0 0 10px var(--bloom-color);
        } 

        /* —— 页面改成铺满浏览器，不再用 flex 居中 —— */
        body { 
            background-color: #0c0c0c; 
            color: #f8fafc; 
            font-family: 'Noto Serif SC', serif; 
            margin: 0; 
            user-select: none; 
            height: 100vh;
            overflow: hidden;
        } 

        /* —— 游戏容器始终占满整个视口 —— */
        #game-container { 
            position: relative; 
            width: 100vw; 
            height: 100vh;
            max-width: 100vw; 
            max-height: 100vh; 
            display: flex; 
            flex-direction: column; 
            justify-content: flex-end; 
            background-size: cover; 
            background-position: center; 
            background-repeat: no-repeat; 
            transition: background-image 1.5s ease-in-out, filter 1s ease; 
            background-color: #000;
            box-shadow: 0 0 100px rgba(0,0,0,0.8);
            overflow: hidden;
        } 

        /* 加载指示器 */
        #loading-indicator {
            position: absolute;
            top: 20px;
            right: 20px;
            font-size: 0.7rem;
            color: rgba(255,255,255,0.3);
            display: none;
            letter-spacing: 0.2em;
            z-index: 30;
        }

        #intro-overlay {
            position: absolute;
            inset: 0;
            background: #000;
            z-index: 100;
            display: flex;
            align-items: center;
            justify-content: flex-start; 
            padding-left: 12vw;
        }

        .intro-text-container {
            max-width: 50vw;
            text-align: left; 
        }

        .intro-text {
            color: #E5E7EB;
            font-weight: 300;
            font-size: 1.6rem;
            line-height: 2.8;
            letter-spacing: 0.15em;
            white-space: pre-line;
            opacity: 0;
            transition: opacity 1s ease;
        }

        .intro-text.fade-in { opacity: 1; }

        .dialogue-box { 
            position: absolute; 
            bottom: 30px; 
            left: 50%;
            transform: translateX(-50%) translateY(150%);
            width: 85%; 
            max-width: 1200px;
            z-index: 20; 
            background: rgba(25, 25, 25, 0.4); 
            backdrop-filter: blur(30px) saturate(180%); 
            -webkit-backdrop-filter: blur(30px) saturate(180%);
            padding: 1.5rem 3rem; 
            min-height: 120px;
            height: auto; 
            cursor: pointer; 
            border: 1px solid rgba(255, 255, 255, 0.12); 
            border-radius: 24px;
            display: flex;
            flex-direction: column;
            justify-content: center;
            transition: transform 1.2s cubic-bezier(0.19, 1, 0.22, 1), opacity 0.8s;
            opacity: 0;
            box-shadow: 0 20px 50px rgba(0,0,0,0.4);
        } 

        .dialogue-box.visible {
            transform: translateX(-50%) translateY(0);
            opacity: 1;
        }

        #character-name {
            font-size: 0.8rem;
            letter-spacing: 0.4em;
            margin-bottom: 0.6rem;
            color: var(--tech-blue);
            font-weight: 700;
            text-transform: uppercase;
            opacity: 0.9;
        }

        #dialogue-text {
            color: rgba(255, 255, 255, 0.92);
            letter-spacing: 0.05rem;
            line-height: 1.8;
            font-size: 1.15rem;
            font-weight: 300;
        }

        #continue-hint {
            position: absolute;
            bottom: 1rem;
            right: 2rem;
            font-size: 0.6rem;
            color: rgba(255, 255, 255, 0.25);
            letter-spacing: 0.2em;
        }

        .inner-thought { font-style: italic; color: #cbd5e1; } 

        .choice-container { 
            position: absolute; 
            top: 45%; 
            left: 50%; 
            transform: translate(-50%, -50%); 
            display: flex; 
            flex-direction: column; 
            align-items: center;
            justify-content: center;
            gap: 1.2rem; 
            z-index: 50; 
            width: 80%;
            max-width: 620px; 
        } 

        .choice-btn { 
            width: 100%;
            padding: 1.25rem 2.5rem; 
            background: rgba(255, 255, 255, 0.08); 
            backdrop-filter: blur(25px) saturate(160%);
            -webkit-backdrop-filter: blur(25px) saturate(160%);
            border: 1px solid rgba(255, 255, 255, 0.15);
            border-radius: 20px;
            color: #ffffff; 
            cursor: pointer; 
            transition: all 0.4s cubic-bezier(0.25, 0.46, 0.45, 0.94); 
            text-align: center; 
            font-size: 1.05rem; 
            letter-spacing: 0.1rem;
            box-shadow: 0 4px 15px rgba(0,0,0,0.1);
            position: relative;
            overflow: hidden;
        } 

        .choice-btn:hover { 
            background: rgba(255, 255, 255, 0.15); 
            transform: scale(1.02);
            border-color: rgba(255, 255, 255, 0.3);
            box-shadow: 0 8px 30px rgba(0,0,0,0.2);
        } 
        
        #epilogue-overlay {
            position: absolute;
            inset: 0;
            background: #000;
            z-index: 100;
            display: none;
            align-items: center;
            justify-content: center;
        }

        .epilogue-content {
            width: 100%;
            max-width: 1600px;
            height: 100%;
            display: flex;
            align-items: center;
            justify-content: flex-start;
            padding-left: 15%;
            color: #E5E7EB; 
            opacity: 0;
            transition: opacity 1s ease;
        }

        #epilogue-text-box {
            width: 40%;
            font-size: 21px; 
            line-height: 1.75; 
            text-align: left;
            text-shadow: 0 0 8px rgba(255, 255, 255, 0.15); 
            white-space: pre-line;
            font-weight: 300;
            letter-spacing: 0.1em;
        }

        .flash { position: absolute; inset: 0; background: white; z-index: 100; display: none; }
        .flash-red { position: absolute; inset: 0; background: #991b1b; z-index: 100; display: none; opacity: 0.3; }

        .fade-in-anim { animation: fadeIn 1s forwards cubic-bezier(0.19, 1, 0.22, 1); }
        @keyframes fadeIn { 
            from { opacity: 0; transform: translateY(10px); filter: blur(10px); } 
            to { opacity: 1; transform: translateY(0); filter: blur(0); } 
        }

        /* 小屏简单适配（可选） */
        @media (max-width: 768px) {
            .dialogue-box {
                bottom: 10px;
                width: 92%;
                padding: 1rem 1.4rem;
            }
        }
    </style> 
</head> 
<body> 

    <div id="game-container"> 
        <div id="loading-indicator">SYNCING REALITY...</div>
        <div id="flash-effect" class="flash"></div> 
        <div id="flash-red-effect" class="flash-red"></div> 

        <div id="intro-overlay">
            <div class="intro-text-container">
                <div id="intro-text-element" class="intro-text"></div>
            </div>
        </div>

        <div id="epilogue-overlay">
            <div id="epilogue-text-wrapper" class="epilogue-content">
                <div id="epilogue-text-box"></div>
            </div>
        </div>

        <div id="choice-box" class="choice-container hidden"></div> 

        <div id="dialogue-box" class="dialogue-box" onclick="nextStep()"> 
            <div id="character-name"></div> 
            <div id="dialogue-text"></div> 
            <div id="continue-hint">AWAITING SYNCHRONIZATION...</div> 
        </div> 
    </div>

    <script> 
        const state = { 
            step: 0, 
            typing: false,
            typingTimer: null,
            currentHTML: "",
            isIntroActive: true,
            isEnding: false,
            isChangingScene: false
        }; 

        const scenes = [
            /* 0 */ { name: "我", text: "这里是041号泡泡。科学家说，我们生活在人类文明最后的琥珀里。一切都是恒温、恒湿、恒久的宁静。", bg: "bg00.jpg" },
            /* 1 */ { text: "在这个直径仅有十公里的球形城市里，每一片叶子的颜色都被预先设定为最令人舒适的嫩绿。气温永远是适宜的26度。", thought: true, bg: "bg01.5.jpg" },
            /* 2 */ { text: "空气中弥漫着‘02号人造薰衣草’的味道。这种分子能够精准地抑制大脑杏仁核的焦虑反应。", thought: true, bg: "bg02.jpg" },
            /* 3 */ { text: "但我为什么，总觉得，那种温润的幸福，仿佛一层厚重的塑料膜，隔绝了所有的真实。", thought: true, bg: "bg03.jpg" },
            /* 4 */ { name: "改变", text: "直到那天，我偶然走入了图书馆那个被禁止进入的角落。那里堆满了不再被扫描的、被时代遗弃的实体杂物。", thought: true, bg: "bg04.jpg" },
            /* 5 */ { text: "在一排排冰冷的全息接口中，我摸到了一个厚重的、粗糙的、带着霉味的东西。那是一本真正的纸质书。", thought: true, bg: "bg05.jpg" },
            /* 6 */ { text: "我的手在颤抖。这种触感……就像是某种古老生命的低语。当我指尖划过封面，感觉到一种从未有过的颤栗。", thought: true, bg: "bg06.jpg" },
            /* 7 */ { text: "翻开第一页，上面写着：‘在迷宫中搜寻比停留在没有奶酪的地方更安全。’", thought: true, bg: "bg07.jpg" },
            /* 8 */ { text: "就在我读完这句话的瞬间，我的视野边缘跳出了刺眼的警报文字。我的生物监测器在疯狂跳动。", action: () => flashRed(), bg: "bg08.jpg" },
            /* 9 */ { name: "中央系统", text: "【警报：0591号公民。检测到您的情绪指标剧烈波动。请立即闭上眼，默诵《公民幸福准则》。】", bg: "bg08.jpg" },
            /* 10 */ { text: "沉重的脚步声从走廊尽头传来。我必须在几秒钟内做出决定，这决定将彻底改变我的人生轨迹。", action: () => {
                showChoices([
                    { text: "【放弃觉醒】：将书交还给系统。将这个秘密埋藏心中不再提起。", next: 11 },
                    { text: "【保留火种】：将书藏入怀中。我想看看这个梦的边缘。", next: 12 }
                ]);
            }, bg: "bg08.jpg" },
            /* 11 */ { text: "我闭上了眼睛，深深吸入了温柔的薰衣草香气。当管理者微笑着走来时，我主动递出了书。后来，我被带去消除了记忆。安适纪元101年，我去世了，我死得很开心，但也从未活过。", action: () => triggerEnding("结局：温顺的标本"), bg: "bg11.jpg" },
            /* 12 */ { name: "我的房间", text: "我攥紧了那本书。这是我这辈子第一次如此真切地感受到自己有一颗心脏在跳动。", action: () => flash(), bg: "bg12.jpg" },
            /* 13 */ { text: "我在书缝里发现了一个带有切面的透明晶体——三棱镜。", thought: true, bg: "bg13.jpg" },
            /* 14 */ { text: "午间11:59，是系统重置的时间，如果想要知道这一切的真相，就看看吧。", thought: true, bg: "bg13.jpg" },
            /* 15 */ { name: "我的房间", text: "午夜 12:00，我举起三棱镜，对准了那个巨大的，弧形的泡泡。", action: () => { document.getElementById('game-container').style.filter = "contrast(1.2) saturate(1.5)"; }, bg: "bg15.jpg" },
            /* 16 */ { text: "在那一秒钟，全息投影骤阵消失。我看到了真正的世界。那是疯狂生长的、深绿色的森林。", action: () => { document.getElementById('game-container').style.filter = "none"; }, bg: "bg15.jpg" },
            /* 17 */ { name: "我", text: "这一晚，我无法入眠。每一次闭眼，我都能看到那种能吞噬灵魂的深绿。", bg: "bg17.jpg" },
            /* 18 */ { name: "餐厅", text: "早餐时，我看着父母。他们机械地切割着人造合成肉。我试探着问：‘爸爸，有人去过泡泡外面吗？’", bg: "bg18.jpg" },
            /* 19 */ { name: "父亲", text: "‘好像在你爷爷的那会，有一对情侣跑出去了，管理者说他们一出去就被强酸腐蚀了。’", bg: "bg19.jpg" },
            /* 20 */ { text: "接下来的这几天，我越来越发现这个世界的不正常。隔壁的邻居每天早晨八点，都在修剪灌木。在同一个位置剪下同一根枝条，12:00 剪下的树枝又瞬间长了出来。每只蝴蝶，扇动翅膀的幅度和飞行轨迹都是如出一辙。", thought: true, bg: "bg20.jpg" },
            /* 21 */ { name: "抉择", text: "我不能再待在这里了，这种‘幸福’正在像慢性毒药一样毒死我的灵魂。我该如何寻找出口？", action: () => {
                showChoices([
                    { text: "【潜入中央实验室】：寻找城市的结构图，搞清楚这个泡泡的边界。", next: 22 },
                    { text: "【寻找排放系统】：最肮脏的地方，往往是通往自由的捷径。", next: 23 }
                ]);
            }, bg: "bg21.jpg" },
            /* 22 */ { text: "实验室的墙壁上写满了数据。我没找到图纸，却发现了泡泡的能源核心——它正在枯竭。管理者隐瞒了真相。", next: 24, bg: "bg22.jpg" },
            /* 23 */ { text: "下水道里没有薰衣草的味道。只有在那腐烂、潮湿的地方，我才感觉到自己是真的活着。", next: 24, bg: "bg23.jpg" },
            /* 24 */ { name: "荒野边缘", text: "我终于爬出了那个令人窒息的排水管。第一口真正的空气灌入肺部，带着泥土的苦味，像刀割一样。我疯狂地咳嗽，却在笑。", action: () => { flash(); }, bg: "bg24.jpg" },
            /* 25 */ { name: "我", text: "这是书上从未描述过的世界，曾经的城市被浓郁的绿色吞没，建筑在藤蔓与树根中倾斜、生长、崩解。文明在这里破败，却仍然站立。空气中没有香精，阳光落在皮肤上，带着真实的温度。", bg: "bg25.jpg" },
            /* 26 */ { name: "废墟大楼", text: "我看到了一个皮鞋脚印，不是泡泡中穿的软底鞋，这种鞋印在泥土上留下了深刻的凹槽。", bg: "bg26.jpg" },
            /* 27 */ { name: "废墟大楼", text: "我跟着地上那串已经干涸、风化了数十年的皮鞋脚印，爬上了一座支离破碎的废弃摩天大楼。", bg: "bg27.jpg" },
            /* 28 */ { name: "顶层", text: "在那风声咆哮的顶层，我看到了一具守在窗前的骷髅。他穿着皮衣，戴着开裂的皮帽。他保持着那个姿势几十年了，仿佛在等待着什么。", bg: "bg28.jpg" },
            /* 29 */ { name: "书桌", text: "他干枯的手边摆着另一本书。封面上写着《真实物种名录》。", bg: "bg29.jpg" },
            /* 30 */ { text: "书里记录着泡泡外的一切：火柴、远古食物、还有种子。我摸索着他的行囊，竟然找到了一些火柴。", thought: true, bg: "bg30.jpg" },
            /* 31 */ { text: "在书的最后一页，我发现了一张泛黄的照片。一对年轻的情侣在绿色的草地上大笑，而图中女主的服装正是这具骷髅所穿的衣服。那个男人……怎么感觉有点眼熟？", thought: true, bg: "bg31.jpg" },
            /* 32 */ { name: "我", text: "我不再满足于在泡泡边境探索，开始在两个世界之间穿梭。我像是一个窃贼，从荒野偷取名为‘真实’的赃物，带回那个虚假的琥珀里。", bg: "bg32.jpg" },
            /* 33 */ { name: "秘密角落", text: "根据先行者留下的笔记，我在泡泡的角落种下了一颗种子。我用那根火柴，在深夜点燃了一簇真实的火焰。看着那跳动的火苗，感觉看到了生命的活性。", thought: true, bg: "bg33.jpg" },
            /* 34 */ { name: "废墟电波", text: "一天，我在荒野的废墟中，修好了一台古老的广播机。里面传出刺耳的沙沙声。", bg: "bg34.jpg" },
            /* 35 */ { name: "未知频率", text: "‘……坐标 42.88……如果你还想看真正的日出……到这来吧。我们……在这里等待幸存的火种。’", bg: "bg34.jpg" },
            /* 36 */ { name: "暴露", text: "我频繁的失踪终究引起了父母的注意。当我这天回到家时，父母站在阴影里，手里死死攥着电话。", bg: "bg36.jpg" },
            /* 37 */ { name: "母亲", text: "（声音颤抖）孩子，不要怪我，我们都是为了你好，外面的世界太危险了……", bg: "bg37.jpg" },
            /* 38 */ { name: "抓捕", text: "管理者破门而入。我以为我会被审判，被消除记忆。但我没有被送往监狱，而是被带到了泡泡的核心圣殿——城市的最高点。", bg: "bg38.jpg" },
            /* 39 */ { name: "管理者", text: "一个老者转过身， his 眼神深邃而悲哀。是他？！虽然苍老了几分，但还是能看得出来，他正是照片上的那个男人。", bg: "bg39.jpg" },
            /* 40 */ { name: "管理者", text: "‘你以为走出去就是勇敢吗？我见过那些所谓的勇敢者：冻死、饿死、互相撕咬，然后把这叫做自由。’", bg: "bg40.jpg" },
            /* 41 */ { name: "管理者", text: "‘你很聪明，留在这里吧，继承我的位置。你会有花不完的钱，吃不完的美食，掌控整个世界的权利。我们可以一起为人类编织更美的梦。为什么要让他们面对绝望？我们是在保护他们。’", bg: "bg41.jpg" },
            /* 42 */ { text: "他向我伸出了苍老的手。我能感觉到我的心脏在疯狂搏动。", action: () => {
                showChoices([
                    { text: "【继承美梦】：成为这个伟大谎言的守护者。", next: 43 },
                    { text: "【冲向真实】：推开一切。人类不应该死在培养皿里！", next: 44 }
                ]);
            }, bg: "bg42.jpg" },
            /* 43 */ { text: "我接过了钥匙。亲手封死了那个通往真实世界的下水口。我从此生活在高塔之上，为所有人调制薰衣草香。我沉浸在父母欣慰的表情，人民的欢呼中。我知道我在说谎，但我告诉自己，这是为了大局。", action: async () => {
                await waitTyping();
                await new Promise(r => setTimeout(r, 2000));
                triggerEnding("结局：继承人");
            }, bg: "bg43.jpg" },
            /* 44 */ { name: "我", text: "你不是选择了这里，你只是习惯了这里！你背叛了她，也背叛了你自己！", bg: "bg44.jpg" },
            /* 45 */ { name: "冲突", text: "我猛地推开了那个老人，撞开了看守。", bg: "bg45.jpg" },
            /* 46 */ { name: "逃离高塔", text: "我沿着无尽的旋转楼梯下冲。身后的警报声如同嘶吼。我必须在系统封死出口前，跑完这最后的几千米！", bg: "bg46.jpg" },
            /* 47 */ { name: "女孩", text: "在出口的街道转角，我停住了。一个小女孩，正用一种迷茫的眼神观察着全息蝴蝶。", bg: "bg47.jpg" },
            /* 48 */ { name: "女孩的声音", text: "‘小女孩和朋友说道：“你发现了吗，这个蝴蝶运动的轨迹总是一样的。”’", bg: "bg48.jpg" },
            /* 49 */ { name: "抉择时刻", text: "我看着她，仿佛看到了曾经的自己。", action: () => {
                showChoices([
                    { text: "【留下火种】：将笔记撕下一页，折成纸飞机扔向她。", next: 50 },
                    { text: "【独自奔向荒野】：让她继续活在编织的美好泡泡中吧。", next: 51 }
                ]);
            }, bg: "bg49.jpg" },
            /* 50 */ { text: "纸飞机划出白色的弧线，落在了女孩的脚边。我没有回头，纵身跃入了黑暗通道。", next: 52, bg: "bg50.jpg" },
            /* 51 */ { text: "我选择了沉默。我最后看了一眼这蓝色的牢笼，决然地钻进了通往灰色的甬道。", next: 52, bg: "bg51.jpg" },
            /* 52 */ { name: "我", text: "荒野的风再次割裂了我的外衣。我站在十字路口，未来就在我脚下。", bg: "bg52.jpg" },
            /* 53 */ { text: "前面有两条路：回到废墟大楼守望可能觉醒的人；或者去寻找广播里的同类。", action: () => {
                showChoices([
                    { text: "【成为荒野守望者】：我相信，有些人是可以被拯救的。", next: 54 },
                    { text: "【寻找新生之旅】：向坐标 42.88 进发。", next: 60 }
                ]);
            }, bg: "bg53.jpg" },
            /* 54 */ { name: "先行者", text: "我回到了那座熟悉的废墟。穿上了先行者的皮衣，坐在她曾经坐过的窗边，看着远方那个白色的、虚假的巨球。", bg: "bg54.jpg" },
            /* 55 */ { name: "先行者", text: "你学着先行者的样子，开始记录真实世界的一切，并把这些真相叠成纸飞机，送回泡泡里，盼望着有人能醒来。", bg: "bg55.jpg" },
            /* 56 */ { name: "先行者", text: "几十年过去了。你老了，头发花白。", bg: "bg56.jpg" },
            /* 57 */ { name: "先行者", text: "阳光灿烂的一个午后，泡泡的排污口传来了一声熟悉的金属摩擦声。一个满身泥土、手里紧紧攥着你当年留下那本旧绘本的女孩，跌跌撞撞地爬了出来，瞪大眼睛看着面前的新世界。", bg: "bg57.jpg" },
            /* 58 */ { name: "先行者", text: "你站起身，拿出一双修补好的硬底靴，微笑着走向那个不知所措的女孩。", bg: "bg58.jpg" },
            /* 59 */ { name: "我", text: "路很长，但是你终于醒了。", action: async () => {
                await waitTyping();
                await new Promise(r => setTimeout(r, 2000));
                triggerEnding("结局：先行者");
            }, bg: "bg59.jpg" },
            /* 60 */ { name: "征程", text: "我选择了出发。这不仅仅是地理上的跨越，更是灵魂的远征。", bg: "bg60.jpg" },
            /* 61 */ { name: "我", text: "在坐标点的篝火旁，老农夫向我走来：‘欢迎来到这里，先行者。你是从041号泡泡出来的吗？’", bg: "bg61.jpg" },
            /* 62 */ { name: "我", text: "我愣了一下，下意识地回答：‘我是0591号。’老农夫哈哈大笑：‘那是编号。我是问你的真实名字，你叫什么名字？’", bg: "bg62.jpg" },
            /* 63 */ { text: "我看着金色的夕阳，露出了一生中最灿烂的微笑：‘我的名字叫……Liusheng。为了庆祝，我终于找到了我的新生。’", action: async () => {
                await waitTyping();
                await new Promise(r => setTimeout(r, 2500));
                triggerEnding("结局：新生");
            }, bg: "bg63.jpg" }
        ];

        // 等待打字机完成
        function waitTyping() {
            return new Promise(resolve => {
                const check = setInterval(() => {
                    if (!state.typing) { clearInterval(check); resolve(); }
                }, 100);
            });
        }

        // 更安全的图片预加载：先绑事件，再设 src
        function preloadImage(url) {
            return new Promise((resolve) => {
                const img = new Image();
                img.onload = () => resolve(true);
                img.onerror = () => resolve(false); // 出错也要 resolve，避免卡死
                img.src = url;
            });
        }

        // 场景渲染
        async function renderScene() {
            if (state.isIntroActive || state.isEnding) return;
            const scene = scenes[state.step];
            if (!scene) return;
            
            state.isChangingScene = true;

            const loading = document.getElementById('loading-indicator');
            const container = document.getElementById('game-container');
            const box = document.getElementById('dialogue-box');
            const nameEl = document.getElementById('character-name');
            const textEl = document.getElementById('dialogue-text');

            try {
                if (scene.bg) {
                    loading.style.display = 'block';
                    await preloadImage(scene.bg);
                    container.style.backgroundImage = `url('${scene.bg}')`;
                    loading.style.display = 'none';
                }
                
                box.classList.add('visible');
                
                nameEl.innerText = scene.name || "";
                state.currentHTML = scene.thought ? `<span class="inner-thought">${scene.text}</span>` : scene.text;
                
                // 图片确定在缓存后，再启动打字机
                typeWriter(textEl, state.currentHTML, 40);
                
                if (typeof scene.action === 'function') scene.action();
            } finally {
                state.isChangingScene = false;
            }
        }

        // 下一步逻辑：先快进，再换场景
        async function nextStep() {
            if (state.isIntroActive || state.isEnding || state.isChangingScene) return;
            
            const textEl = document.getElementById('dialogue-text');

            if (state.typing) {
                clearInterval(state.typingTimer);
                textEl.innerHTML = state.currentHTML;
                state.typing = false;
                return;
            }
            
            if (!document.getElementById('choice-box').classList.contains('hidden')) return;
            
            const currentScene = scenes[state.step];
            if (currentScene && typeof currentScene.next === 'number') {
                state.step = currentScene.next;
            } else {
                state.step++;
            }
            
            if (state.step < scenes.length) {
                await renderScene();
            }
        }

        // 结局处理逻辑
        async function triggerEnding(title) {
            state.isEnding = true;
            const dialogueBox = document.getElementById('dialogue-box');
            dialogueBox.classList.remove('visible');
            dialogueBox.style.opacity = "0";
            document.getElementById('game-container').style.backgroundImage = "none";
            document.getElementById('game-container').style.backgroundColor = "#000";
            
            await new Promise(r => setTimeout(r, 1500));
            
            if (title === "结局：新生") { await playEpilogue(); return; } 
            else if (title === "结局：先行者") { await playWatcherEpilogue(); return; } 
            else if (title === "结局：继承人") {
                dialogueBox.style.opacity = "1";
                dialogueBox.classList.add('visible');
                dialogueBox.style.height = "100vh";
                dialogueBox.style.width = "100vw";
                dialogueBox.style.borderRadius = "0";
                dialogueBox.style.bottom = "0";
                dialogueBox.style.background = "#000";
                document.getElementById('character-name').innerText = "SYSTEM: SIMULATION END";
                document.getElementById('dialogue-text').innerHTML = `<div class="text-center mt-20"><span class="text-3xl tracking-[0.6em] text-white uppercase">${title}</span></div>`;
                document.getElementById('continue-hint').style.display = 'none';
                
                await new Promise(r => setTimeout(r, 3000));
                dialogueBox.classList.remove('visible');
                dialogueBox.style.opacity = "0";
                await new Promise(r => setTimeout(r, 1000));
                await playInheritorEpilogue();
                return;
            }

            dialogueBox.style.opacity = "1";
            dialogueBox.classList.add('visible');
            dialogueBox.style.height = "100vh";
            dialogueBox.style.width = "100vw";
            dialogueBox.style.borderRadius = "0";
            dialogueBox.style.bottom = "0";
            dialogueBox.style.background = "#000";
            document.getElementById('character-name').innerText = "SYSTEM: SIMULATION END";
            document.getElementById('dialogue-text').innerHTML = `<div class="text-center mt-20"><span class="text-3xl tracking-[0.6em] text-white uppercase">${title}</span></div>`;
            document.getElementById('continue-hint').style.display = 'none';
        }

        async function playInheritorEpilogue() {
            const overlay = document.getElementById('epilogue-overlay');
            const wrapper = document.getElementById('epilogue-text-wrapper');
            const textBox = document.getElementById('epilogue-text-box');
            overlay.style.display = 'flex';
            const pages = [
                { text: `我留了下来。\n\n在泡泡之中，\n我不必再为生存费力，\n也不必为选择负责。`, stay: 3500 },
                { text: `他们称之为奖励，\n称之为责任。\n我好像也有了不错的收获。\n\n而这些，\n恰好足够让我\n心安理得地停下脚步。`, stay: 4000 },
                { text: `但也让我慢慢忘记——\n自己曾经想要走出去。`, stay: 5000 }
            ];
            for (let i = 0; i < pages.length; i++) {
                textBox.innerText = pages[i].text;
                wrapper.style.opacity = "1";
                await new Promise(r => setTimeout(r, 1000 + pages[i].stay));
                wrapper.style.opacity = "0";
                await new Promise(r => setTimeout(r, 800));
            }
            await new Promise(r => setTimeout(r, 2000));
            wrapper.style.justifyContent = "center";
            wrapper.style.paddingLeft = "0";
            textBox.style.width = "100%";
            textBox.style.textAlign = "center";
            textBox.style.letterSpacing = "1em";
            textBox.innerHTML = `感 谢 游 玩`;
            wrapper.style.opacity = "1";
        }

        async function playWatcherEpilogue() {
            const overlay = document.getElementById('epilogue-overlay');
            const wrapper = document.getElementById('epilogue-text-wrapper');
            const textBox = document.getElementById('epilogue-text-box');
            overlay.style.display = 'flex';
            const pages = [
                { text: `走出泡泡之后，我才意识到，离开并不是唯一的选择。\n\n并不是每个人都需要走得很远，也不是每个人都必须立刻改变生活的方向。`, stay: 3500 },
                { text: `我依然生活在这个世界里，走在熟悉的街道上，做着普通的事情。\n\n世界看起来并没有发生巨大的变化，但我知道，我已经不再完全住在泡泡里了。`, stay: 4000 },
                { text: `如果有人愿意走出来，我希望他们知道：\n\n不需要走得很远，只要迈出泡泡的边界，就已经足够勇敢。`, stay: 5000 },
                { text: `“路很长，但你终于醒了。”`, stay: 6000, isHighlight: true }
            ];
            for (let i = 0; i < pages.length; i++) {
                if (pages[i].isHighlight) { textBox.style.fontSize = "32px"; textBox.style.letterSpacing = "0.5em"; textBox.style.color = "#FFF"; }
                textBox.innerText = pages[i].text;
                wrapper.style.opacity = "1";
                await new Promise(r => setTimeout(r, pages[i].stay)); 
                wrapper.style.opacity = "0";
                await new Promise(r => setTimeout(r, 2000)); 
            }
            textBox.style.fontSize = "1.5rem";
            textBox.style.letterSpacing = "1em";
            textBox.style.textAlign = "center";
            textBox.style.width = "100%";
            wrapper.style.justifyContent = "center";
            wrapper.style.paddingLeft = "0";
            textBox.innerHTML = `感 谢 游 玩`;
            wrapper.style.opacity = "1";
        }

        async function playEpilogue() {
            const overlay = document.getElementById('epilogue-overlay');
            const wrapper = document.getElementById('epilogue-text-wrapper');
            const textBox = document.getElementById('epilogue-text-box');
            overlay.style.display = 'flex';
            const pages = [
                { text: `在生活中，我们经常住在一个\n属于自己的“泡泡”里。\n\n那里真的很安适、很舒服，\n却也在不知不觉中，\n让我们慢慢远离了真实的自己，\n以及正在发生的生活。`, stay: 3500 },
                { text: `如果在泡泡里停留得太久，\n它就不再只是避风的地方，\n而会变成滋养惰性的温床。\n\n我们也许获得了暂时的舒适，\n却是以透支未来为代价的。`, stay: 3500 },
                { text: `如果一直停留在原地，\n世界就只会停留在\n我们已经知道的样子。\n\n所以，\n所谓“活在当下”，\n也可能只是被困在当下。`, stay: 4500 },
                { text: `别害怕，\n\n走出去。`, stay: 5000 }
            ];
            for (let i = 0; i < pages.length; i++) {
                textBox.innerText = pages[i].text;
                wrapper.style.opacity = "1";
                await new Promise(r => setTimeout(r, 1000 + pages[i].stay));
                wrapper.style.opacity = "0";
                await new Promise(r => setTimeout(r, 1500)); 
            }
            textBox.style.textAlign = "center";
            textBox.style.width = "100%";
            wrapper.style.justifyContent = "center";
            wrapper.style.paddingLeft = "0";
            textBox.innerHTML = `感 谢 游 玩`;
            wrapper.style.opacity = "1";
        }

        function typeWriter(el, html, speed) {
            if (state.typingTimer) clearInterval(state.typingTimer);
            state.typing = true; 
            el.innerHTML = "";
            let i = 0;
            const isThought = html.includes('inner-thought');
            const plainText = html.replace(/<[^>]*>/g, '');
            state.typingTimer = setInterval(() => {
                if (i < plainText.length) {
                    const current = plainText.substring(0, i + 1);
                    el.innerHTML = isThought ? `<span class="inner-thought">${current}</span>` : current;
                    i++;
                } else {
                    clearInterval(state.typingTimer);
                    state.typing = false;
                }
            }, speed);
        }

        function showChoices(choices) {
            const box = document.getElementById('choice-box');
            box.innerHTML = '';
            box.classList.remove('hidden');
            document.getElementById('dialogue-box').style.opacity = "0.3";
            document.getElementById('dialogue-box').style.pointerEvents = "none";

            choices.forEach(c => {
                const btn = document.createElement('div');
                btn.className = 'choice-btn fade-in-anim';
                btn.innerText = c.text;
                btn.onclick = (e) => {
                    e.stopPropagation();
                    box.classList.add('hidden');
                    document.getElementById('dialogue-box').style.opacity = "1";
                    document.getElementById('dialogue-box').style.pointerEvents = "auto";
                    state.step = c.next;
                    renderScene();
                };
                box.appendChild(btn);
            });
        }

        function runIntro() {
            const introEl = document.getElementById('intro-text-element');
            const introOverlay = document.getElementById('intro-overlay');
            const content = [
                { text: `安适纪元 19 年。\n\n在人类长期回避现实过程中，\n气候系统逐步失衡，世界由此走向终结。`, stay: 3500 },
                { text: `管理者启动了「泡泡计划」，\n将幸存者安置进封闭的球形城市。`, stay: 3000 },
                { text: `世界因此维持在一种稳定而安适的状态之中。`, stay: 2500 }
            ];
            
            let currentPage = 0;
            function playPage() {
                if (currentPage >= content.length) {
                    setTimeout(() => {
                        introOverlay.style.display = 'none';
                        state.isIntroActive = false;
                        renderScene();
                    }, 1000);
                    return;
                }
                const page = content[currentPage];
                introEl.innerText = page.text;
                introEl.className = 'intro-text'; 
                setTimeout(() => introEl.classList.add('fade-in'), 50);
                
                setTimeout(() => {
                    introEl.classList.remove('fade-in');
                    setTimeout(() => {
                        currentPage++;
                        playPage();
                    }, 800);
                }, 1000 + page.stay);
            }
            playPage();
        }
        
        function flash() {
            const f = document.getElementById('flash-effect');
            f.style.display='block';
            setTimeout(()=>f.style.display='none', 600);
        }

        function flashRed() {
            const f = document.getElementById('flash-red-effect');
            f.style.display='block';
            setTimeout(()=>f.style.display='none', 800);
        }

        window.onload = runIntro;
    </script> 
</body> 
</html>
