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
