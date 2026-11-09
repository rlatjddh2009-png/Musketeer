<!DOCTYPE html>
<html lang="ko">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>머스킷티어 (Musketeer) - 18세기 결투 보드게임</title>
    <!-- Tailwind CSS CDN -->
    <script src="https://cdn.tailwindcss.com"></script>
    <!-- Google Fonts: 앤틱한 느낌의 폰트 (Cinzel 및 Nanum Myeongjo) -->
    <link href="https://fonts.googleapis.com/css2?family=Cinzel:wght@600;700;900&family=Nanum+Myeongjo:wght@400;700&display=swap" rel="stylesheet">
    <!-- FontAwesome 아이콘 -->
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css">
    <style>
        body {
            font-family: 'Nanum Myeongjo', serif;
            background-color: #1f1712;
            color: #f3ebd8;
        }
        .cinzel {
            font-family: 'Cinzel', serif;
        }
        .parchment-bg {
            background-color: #f4ebd0;
            background-image: radial-gradient(circle, #f9f3e3 0%, #e6d5b8 100%);
            color: #2c221e;
        }
        .wood-panel {
            background-color: #3b271a;
            border: 3px solid #5a3d28;
            box-shadow: inset 0 0 15px rgba(0,0,0,0.6);
        }
        .card-parchment {
            background-color: #fdfaf0;
            border: 2px solid #b89768;
            color: #3a2e2b;
            box-shadow: 0 4px 6px rgba(0,0,0,0.2);
            transition: all 0.2s ease;
        }
        .card-parchment:hover {
            transform: translateY(-4px);
            box-shadow: 0 8px 12px rgba(0,0,0,0.3);
        }
        /* 스크롤바 디자인 */
        ::-webkit-scrollbar {
            width: 6px;
            height: 6px;
        }
        ::-webkit-scrollbar-track {
            background: #2a1f18;
        }
        ::-webkit-scrollbar-thumb {
            background: #6a503d;
            border-radius: 3px;
        }
    </style>
</head>
<body class="h-screen overflow-hidden flex flex-col justify-between select-none">

    <!-- 로비 화면 (초기 화면) -->
    <div id="lobby-screen" class="absolute inset-0 z-50 parchment-bg flex flex-col items-center justify-center p-6 overflow-y-auto">
        <div class="max-w-xl w-full wood-panel p-8 rounded-lg text-[#f3ebd8] shadow-2xl border-4 border-[#8c6d48]">
            <div class="text-center mb-6">
                <h1 class="cinzel text-4xl font-bold tracking-wider text-[#d4af37] mb-2 drop-shadow-md">MUSKETEER</h1>
                <p class="text-sm text-[#d1c2a5] italic">18세기 흑색화약 사수들의 치명적인 결투 보드게임</p>
            </div>

            <!-- 플레이어 수 설정 -->
            <div class="mb-6">
                <label class="block text-sm font-bold mb-2 text-[#d4af37]">참여 인원 선택 (2 ~ 8인)</label>
                <select id="player-count-select" class="w-full bg-[#2a1f18] text-[#f3ebd8] border border-[#8c6d48] rounded p-2 focus:outline-none">
                    <option value="2">2인 결투</option>
                    <option value="3">3인 난투</option>
                    <option value="4" selected>4인 배틀로얄</option>
                    <option value="5">5인 난투</option>
                    <option value="6">6인 난투</option>
                    <option value="7">7인 난투</option>
                    <option value="8">8인 최대 난투</option>
                </select>
            </div>

            <!-- 플레이어 설정 컨테이너 -->
            <div class="mb-6">
                <label class="block text-sm font-bold mb-2 text-[#d4af37]">참가자 닉네임 및 플레이어 설정</label>
                <div id="player-configs" class="space-y-3 max-h-48 overflow-y-auto pr-2">
                    <!-- 동적 생성 -->
                </div>
            </div>

            <!-- 게임 모드 선택 (로컬 / 온라인 준비) -->
            <div class="grid grid-cols-2 gap-4 mb-6">
                <button onclick="startGame(false)" class="bg-[#8c6d48] hover:bg-[#a3825c] text-[#1f1712] font-bold py-3 px-4 rounded shadow transition cinzel text-sm border border-[#d4af37]">
                    <i class="fa-solid fa-users mr-2"></i> 로컬 게임 시작 (Pass & Play)
                </button>
                <button onclick="openOnlineModal()" class="bg-[#5a3d28] hover:bg-[#6e4d34] text-[#d4af37] font-bold py-3 px-4 rounded shadow transition cinzel text-sm border border-[#8c6d48]">
                    <i class="fa-solid fa-globe mr-2"></i> 온라인 멀티플레이
                </button>
            </div>
            
            <div class="text-center text-xs text-[#a89078]">
                <p>규칙: 6단계 장전을 완료하고 1, 3, 6 주사위로 상대를 저격하여 최후의 생존자가 되세요!</p>
            </div>
        </div>
    </div>

    <!-- 온라인 방 생성/참여 모달 -->
    <div id="online-modal" class="fixed inset-0 z-60 bg-black/80 hidden flex items-center justify-center p-4">
        <div class="max-w-md w-full wood-panel p-6 rounded-lg text-[#f3ebd8] border-2 border-[#d4af37]">
            <h2 class="cinzel text-2xl font-bold text-[#d4af37] mb-4 text-center">온라인 대전방</h2>
            <div class="space-y-4">
                <div>
                    <button onclick="createRoom()" class="w-full bg-[#8c6d48] hover:bg-[#a3825c] text-[#1f1712] font-bold py-3 rounded cinzel">
                        방 만들기 (초대 코드 생성)
                    </button>
                </div>
                <div class="border-t border-[#6a503d] pt-4">
                    <label class="block text-sm mb-2 text-[#d4af37]">초대 코드 입력하기</label>
                    <div class="flex gap-2">
                        <input type="text" id="room-code-input" placeholder="방 코드 입력" class="flex-1 bg-[#2a1f18] border border-[#8c6d48] rounded p-2 text-[#f3ebd8] focus:outline-none">
                        <button onclick="joinRoom()" class="bg-[#5a3d28] hover:bg-[#6e4d34] text-[#f3ebd8] font-bold px-4 rounded cinzel border border-[#8c6d48]">입장</button>
                    </div>
                </div>
                <div id="room-info-box" class="hidden bg-[#2a1f18] p-3 rounded border border-[#8c6d48] text-
