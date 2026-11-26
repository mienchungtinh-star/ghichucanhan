<!DOCTYPE html>
<html lang="vi">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>EmeraldNote - Quản Lý Ghi Chú & Nhắc Nhở Cá Nhân</title>
    <!-- Tải Tailwind CSS -->
    <script src="https://cdn.tailwindcss.com"></script>
    <!-- Tải Google Fonts: Inter & Plus Jakarta Sans -->
    <link rel="preconnect" href="https://fonts.googleapis.com">
    <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
    <link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700&family=Plus+Jakarta+Sans:wght@600;700&display=swap" rel="stylesheet">
    <!-- Tải Phosphor Icons -->
    <script src="https://unpkg.com/@phosphor-icons/web"></script>

    <!-- Cấu hình Tailwind mở rộng bảng màu Xanh lá đậm & Hồng Baby -->
    <script>
        tailwind.config = {
            theme: {
                extend: {
                    fontFamily: {
                        sans: ['Inter', 'sans-serif'],
                        display: ['Plus Jakarta Sans', 'sans-serif'],
                    },
                    colors: {
                        forest: {
                            950: '#0F291E',
                            900: '#1B4332', // Xanh lá đậm chủ đạo
                            800: '#24553F',
                            700: '#2D6A4F', // Xanh lá trung bình
                            600: '#40916C',
                            100: '#D8F3DC',
                            50:  '#EDF7F0',
                        },
                        nude: {
                            50:  '#FFF9FC', // Nền thẻ card trắng ánh hồng tinh tế
                            100: '#FCEEF5', // Nền trang chính Hồng Baby dịu mắt
                            200: '#F8E1EE', // Hồng pastel vùng phụ trợ / badge nhẹ
                            300: '#F3C6DE', // Viền hồng phấn mềm mại
                            400: '#F49AC2', // Hồng baby điểm nhấn nổi bật
                            500: '#D96B9C', // Hồng đậm hơn cho số liệu / icon nhấn
                            800: '#6E475B', // Chữ phụ tông nâu hồng trầm dễ đọc
                            900: '#4A2B3B', // Chữ nhấn đậm hài hòa với nền hồng
                        }
                    },
                    boxShadow: {
                        'soft': '0 4px 20px -2px rgba(27, 67, 50, 0.08)',
                        'card': '0 2px 12px -1px rgba(110, 71, 91, 0.07)',
                        'glow': '0 0 20px rgba(244, 154, 194, 0.4)',
                    }
                }
            }
        }
    </script>

    <style>
        /* Tùy chỉnh thanh cuộn thanh lịch tông Hồng Baby & Xanh lá */
        ::-webkit-scrollbar {
            width: 8px;
            height: 8px;
        }
        ::-webkit-scrollbar-track {
            background: #FCEEF5;
        }
        ::-webkit-scrollbar-thumb {
            background: #F49AC2;
            border-radius: 9999px;
        }
        ::-webkit-scrollbar-thumb:hover {
            background: #2D6A4F;
        }

        /* Hiệu ứng kéo thả */
        .note-card.dragging {
            opacity: 0.45;
            transform: scale(0.98);
            border: 2px dashed #1B4332;
        }
        .note-card.drag-over {
            border-top: 3px solid #F49AC2;
        }

        /* Hiệu ứng rung chuông nhắc nhở */
        @keyframes ring {
            0%, 100% { transform: rotate(0deg); }
            20% { transform: rotate(15deg); }
            40% { transform: rotate(-12deg); }
            60% { transform: rotate(8deg); }
            80% { transform: rotate(-5deg); }
        }
        .animate-ring {
            animation: ring 1.2s ease-in-out infinite;
        }

        /* Ẩn input file gốc */
        input[type="file"] {
            display: none;
        }
    </style>
</head>
<body class="bg-nude-100 text-forest-950 min-h-screen flex flex-col selection:bg-nude-400 selection:text-white">

    <!-- ==================== HEADER / THANH ĐIỀU HƯỚNG ==================== -->
    <header class="bg-forest-900 text-nude-100 sticky top-0 z-30 shadow-md border-b border-forest-800">
        <div class="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8 py-3.5 flex flex-wrap items-center justify-between gap-3">
            <!-- Logo & Tên Ứng Dụng -->
            <div class="flex items-center gap-3">
                <div class="w-10 h-10 rounded-xl bg-nude-400 text-forest-950 flex items-center justify-center shadow-inner">
                    <i class="ph-fill ph-notebook text-2xl"></i>
                </div>
                <div>
                    <h1 class="font-display font-bold text-lg sm:text-xl tracking-tight text-nude-50 flex items-center gap-2">
                        EmeraldNote
                        <span class="text-[11px] font-medium px-2 py-0.5 rounded-full bg-forest-700 text-nude-200 border border-forest-600 hidden sm:inline-block">
                            Personal Workspace
                        </span>
                    </h1>
                    <p id="liveClock" class="text-xs text-nude-300 font-medium">Đang tải thời gian...</p>
                </div>
            </div>

            <!-- Tên chủ lưu trú (Chỉnh sửa trực tiếp & tự động lưu LocalStorage) -->
            <div class="flex items-center gap-2 bg-forest-800/90 border border-forest-700 px-3 py-1.5 rounded-xl shadow-xs">
                <i class="ph-fill ph-user-circle text-nude-400 text-lg shrink-0"></i>
                <label for="ownerNameInput" class="text-[11px] font-medium text-nude-300 whitespace-nowrap hidden md:inline">
                    Chủ lưu trú:
                </label>
                <input type="text" id="ownerNameInput" onchange="saveOwnerName(this.value)" onblur="saveOwnerName(this.value)"
                    placeholder="Nhập tên chủ lưu trú..." title="Bấm để đổi tên chủ lưu trú"
                    class="bg-transparent text-xs sm:text-sm font-semibold text-nude-50 placeholder-nude-300/60 focus:outline-none focus:ring-1 focus:ring-nude-400 rounded px-1.5 py-0.5 w-32 sm:w-44 transition">
            </div>

            <!-- Các công cụ nhanh trên Header -->
            <div class="flex items-center gap-2 sm:gap-3">
                <!-- Nút bật thông báo trình duyệt -->
                <button id="btnRequestNotif" onclick="requestBrowserNotification()" title="Bật thông báo đẩy trên trình duyệt"
                    class="flex items-center gap-1.5 px-3 py-2 rounded-xl text-xs sm:text-sm font-medium bg-forest-800 hover:bg-forest-700 text-nude-100 border border-forest-700 transition cursor-pointer">
                    <i id="notifIcon" class="ph ph-bell text-base text-nude-400"></i>
                    <span id="notifStatusText" class="hidden md:inline">Bật thông báo</span>
                </button>

                <!-- Xuất Backup JSON -->
                <button onclick="exportDataJSON()" title="Tải xuống bản sao lưu JSON"
                    class="flex items-center gap-1.5 px-3 py-2 rounded-xl text-xs sm:text-sm font-medium bg-forest-800 hover:bg-forest-700 text-nude-100 border border-forest-700 transition cursor-pointer">
                    <i class="ph ph-export text-base text-nude-400"></i>
                    <span class="hidden sm:inline">Xuất JSON</span>
                </button>

                <!-- Nhập Backup JSON -->
                <label for="importJsonInput" title="Khôi phục dữ liệu từ file JSON"
                    class="flex items-center gap-1.5 px-3 py-2 rounded-xl text-xs sm:text-sm font-medium bg-nude-400 hover:bg-nude-500 text-forest-950 font-semibold transition cursor-pointer shadow-sm">
                    <i class="ph ph-upload-simple text-base"></i>
                    <span class="hidden sm:inline">Nhập JSON</span>
                </label>
                <input type="file" id="importJsonInput" accept=".json" onchange="importDataJSON(event)">
            </div>
        </div>
    </header>

    <!-- ==================== BANNER NHẮC NHỞ ĐANG KÍCH HOẠT (NẾU CÓ) ==================== -->
    <div id="activeAlarmBanner" class="hidden bg-nude-400 text-forest-950 border-b-2 border-forest-900 shadow-lg transition-all">
        <div class="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8 py-3 flex flex-wrap items-center justify-between gap-3">
            <div class="flex items-center gap-3">
                <div class="w-9 h-9 rounded-full bg-forest-900 text-nude-100 flex items-center justify-center animate-ring shrink-0">
                    <i class="ph-fill ph-bell-ringing text-xl"></i>
                </div>
                <div>
                    <p class="text-xs font-bold uppercase tracking-wider text-forest-900">Đến hạn nhắc nhở!</p>
                    <p id="activeAlarmTitle" class="font-bold text-sm sm:text-base"></p>
                </div>
            </div>
            <div class="flex items-center gap-2">
                <button onclick="completeActiveAlarm()" class="px-3.5 py-1.5 rounded-lg bg-forest-900 hover:bg-forest-800 text-nude-50 text-xs sm:text-sm font-semibold transition cursor-pointer">
                    Đánh dấu xong
                </button>
                <button onclick="dismissActiveAlarm()" class="px-3.5 py-1.5 rounded-lg bg-nude-100 hover:bg-white text-forest-950 text-xs sm:text-sm font-semibold transition cursor-pointer">
                    Đã hiểu
                </button>
            </div>
        </div>
    </div>

    <!-- ==================== NỘI DUNG CHÍNH ==================== -->
    <main class="flex-1 max-w-7xl w-full mx-auto px-4 sm:px-6 lg:px-8 py-6 sm:py-8">

        <!-- KHU VỰC NHẮC NHỞ HÔM NAY / SẮP TỚI -->
        <section id="upcomingRemindersSection" class="mb-6 bg-nude-50 border border-nude-300 rounded-2xl p-4 sm:p-5 shadow-soft">
            <div class="flex flex-wrap items-center justify-between gap-2 mb-3">
                <div class="flex items-center gap-2">
                    <span class="w-8 h-8 rounded-lg bg-forest-900 text-nude-400 flex items-center justify-center">
                        <i class="ph-fill ph-alarm text-lg"></i>
                    </span>
                    <div>
                        <h2 class="font-display font-bold text-base sm:text-lg text-forest-900">Nhắc nhở hôm nay & Sắp tới</h2>
                        <p class="text-xs text-nude-800">Tự động theo dõi thời gian thực và phát âm báo khi đến giờ</p>
                    </div>
                </div>
                <span id="upcomingCountBadge" class="px-3 py-1 rounded-full text-xs font-semibold bg-forest-100 text-forest-900 border border-forest-600/20">
                    0 lịch hẹn
                </span>
            </div>
            <div id="upcomingRemindersList" class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-3">
                <!-- Danh sách nhắc nhở sắp tới sẽ được render tại đây -->
            </div>
        </section>

        <!-- LAYOUT 2 CỘT: FORM THÊM GHI CHÚ (TRÁI) & DANH SÁCH GHI CHÚ (PHẢI) -->
        <div class="grid grid-cols-1 lg:grid-cols-12 gap-6 items-start">

            <!-- CỘT TRÁI: FORM TẠO GHI CHÚ MỚI (4 CỘT TRÊN DESKTOP) -->
            <aside class="lg:col-span-4 bg-nude-50 border border-nude-300 rounded-2xl p-5 shadow-soft lg:sticky lg:top-24">
                <div class="flex items-center justify-between mb-4 pb-3 border-b border-nude-200">
                    <h2 class="font-display font-bold text-lg text-forest-900 flex items-center gap-2">
                        <i class="ph-fill ph-plus-circle text-forest-700 text-xl"></i>
                        Tạo Ghi Chú Mới
                    </h2>
                    <span class="text-[11px] font-medium text-nude-800 bg-nude-200 px-2.5 py-1 rounded-lg">Auto-save</span>
                </div>

                <form id="addNoteForm" onsubmit="handleCreateNote(event)" class="space-y-4">
                    <!-- Tiêu đề ghi chú -->
                    <div>
                        <label for="noteTitle" class="block text-xs font-semibold uppercase tracking-wider text-forest-900 mb-1.5">
                            Tiêu đề ghi chú <span class="text-red-600">*</span>
                        </label>
                        <input type="text" id="noteTitle" required placeholder="VD: Họp dự án Python Web..."
                            class="w-full px-3.5 py-2.5 rounded-xl bg-white border border-nude-300 text-forest-950 text-sm focus:outline-none focus:ring-2 focus:ring-forest-700 transition">
                    </div>

                    <!-- Nội dung chi tiết -->
                    <div>
                        <label for="noteContent" class="block text-xs font-semibold uppercase tracking-wider text-forest-900 mb-1.5">
                            Nội dung chi tiết
                        </label>
                        <textarea id="noteContent" rows="3" placeholder="Nhập mô tả chi tiết công việc, ý tưởng..."
                            class="w-full px-3.5 py-2.5 rounded-xl bg-white border border-nude-300 text-forest-950 text-sm focus:outline-none focus:ring-2 focus:ring-forest-700 transition resize-y"></textarea>
                    </div>

                    <!-- Ngày đến hạn (Deadline) & Giờ nhắc nhở -->
                    <div class="grid grid-cols-1 sm:grid-cols-2 gap-3">
                        <div>
                            <label for="noteDueDate" class="block text-xs font-semibold uppercase tracking-wider text-forest-900 mb-1.5">
                                Ngày đến hạn <span class="text-red-600">*</span>
                            </label>
                            <input type="date" id="noteDueDate" required
                                class="w-full px-3 py-2.5 rounded-xl bg-white border border-nude-300 text-forest-950 text-sm focus:outline-none focus:ring-2 focus:ring-forest-700 transition">
                            <p id="formattedDatePreview" class="text-[11px] text-forest-700 font-medium mt-1">Deadline: --/--/----</p>
                        </div>

                        <div>
                            <label for="noteReminderTime" class="block text-xs font-semibold uppercase tracking-wider text-forest-900 mb-1.5">
                                Giờ nhắc nhở
                            </label>
                            <input type="time" id="noteReminderTime"
                                class="w-full px-3 py-2.5 rounded-xl bg-white border border-nude-300 text-forest-950 text-sm focus:outline-none focus:ring-2 focus:ring-forest-700 transition">
                            <p class="text-[11px] text-nude-800 mt-1">Để trống nếu không hẹn giờ</p>
                        </div>
                    </div>

                    <!-- Danh mục & Mức độ ưu tiên -->
                    <div class="grid grid-cols-1 sm:grid-cols-2 gap-3">
                        <div>
                            <label for="noteCategory" class="block text-xs font-semibold uppercase tracking-wider text-forest-900 mb-1.5">
                                Danh mục
                            </label>
                            <select id="noteCategory"
                                class="w-full px-3 py-2.5 rounded-xl bg-white border border-nude-300 text-forest-950 text-sm focus:outline-none focus:ring-2 focus:ring-forest-700 transition">
                                <option value="Học tập">📚 Học tập</option>
                                <option value="Lịch học">🎓 Lịch học</option>
                                <option value="Công việc">💼 Công việc</option>
                                <option value="Cá nhân" selected>🌿 Cá nhân</option>
                                <option value="Sức khỏe">🧘 Sức khỏe</option>
                                <option value="Khác">📌 Khác</option>
                            </select>
                        </div>

                        <div>
                            <label for="notePriority" class="block text-xs font-semibold uppercase tracking-wider text-forest-900 mb-1.5">
                                Mức độ ưu tiên
                            </label>
                            <select id="notePriority"
                                class="w-full px-3 py-2.5 rounded-xl bg-white border border-nude-300 text-forest-950 text-sm focus:outline-none focus:ring-2 focus:ring-forest-700 transition">
                                <option value="low">🟢 Thấp</option>
                                <option value="normal" selected>🟡 Bình thường</option>
                                <option value="high">🔴 Cao</option>
                            </select>
                        </div>
                    </div>

                    <!-- Lịch học / Lặp lại theo thứ (Thứ 2 - CN) & Thông tin môn/giảng viên -->
                    <div class="bg-nude-100/80 border border-nude-300 rounded-xl p-3 space-y-2.5">
                        <div class="flex flex-wrap items-center justify-between gap-1.5">
                            <span class="text-xs font-semibold uppercase tracking-wider text-forest-900 flex items-center gap-1.5">
                                <i class="ph-fill ph-graduation-cap text-forest-700 text-sm"></i>
                                Lịch học định kỳ (Thứ 2 - CN)
                            </span>
                            <div class="flex flex-wrap items-center gap-1.5">
                                <button type="button" onclick="fillSampleClassSchedule('it_van')"
                                    class="text-[11px] font-semibold text-forest-800 hover:text-forest-950 underline cursor-pointer">
                                    Ca 3 IT
                                </button>
                                <span class="text-nude-400">•</span>
                                <button type="button" onclick="fillSampleClassSchedule('ita_dieu')"
                                    class="text-[11px] font-semibold text-forest-800 hover:text-forest-950 underline cursor-pointer">
                                    Ca 4 ITA
                                </button>
                                <span class="text-nude-400">•</span>
                                <button type="button" onclick="fillSampleClassSchedule('eng_hung')"
                                    class="text-[11px] font-semibold text-forest-800 hover:text-forest-950 underline cursor-pointer">
                                    Tiếng Anh
                                </button>
                            </div>
                        </div>

                        <!-- Chọn các thứ trong tuần -->
                        <div class="flex flex-wrap gap-1.5" id="createDaysPicker">
                            <label class="cursor-pointer select-none"><input type="checkbox" value="1" class="peer sr-only"><span class="px-2 py-1 rounded-lg text-[11px] font-semibold bg-white border border-nude-300 text-forest-900 peer-checked:bg-forest-900 peer-checked:text-nude-50 peer-checked:border-forest-900 inline-block transition">T2</span></label>
                            <label class="cursor-pointer select-none"><input type="checkbox" value="2" class="peer sr-only"><span class="px-2 py-1 rounded-lg text-[11px] font-semibold bg-white border border-nude-300 text-forest-900 peer-checked:bg-forest-900 peer-checked:text-nude-50 peer-checked:border-forest-900 inline-block transition">T3</span></label>
                            <label class="cursor-pointer select-none"><input type="checkbox" value="3" class="peer sr-only"><span class="px-2 py-1 rounded-lg text-[11px] font-semibold bg-white border border-nude-300 text-forest-900 peer-checked:bg-forest-900 peer-checked:text-nude-50 peer-checked:border-forest-900 inline-block transition">T4</span></label>
                            <label class="cursor-pointer select-none"><input type="checkbox" value="4" class="peer sr-only"><span class="px-2 py-1 rounded-lg text-[11px] font-semibold bg-white border border-nude-300 text-forest-900 peer-checked:bg-forest-900 peer-checked:text-nude-50 peer-checked:border-forest-900 inline-block transition">T5</span></label>
                            <label class="cursor-pointer select-none"><input type="checkbox" value="5" class="peer sr-only"><span class="px-2 py-1 rounded-lg text-[11px] font-semibold bg-white border border-nude-300 text-forest-900 peer-checked:bg-forest-900 peer-checked:text-nude-50 peer-checked:border-forest-900 inline-block transition">T6</span></label>
                            <label class="cursor-pointer select-none"><input type="checkbox" value="6" class="peer sr-only"><span class="px-2 py-1 rounded-lg text-[11px] font-semibold bg-white border border-nude-300 text-forest-900 peer-checked:bg-forest-900 peer-checked:text-nude-50 peer-checked:border-forest-900 inline-block transition">T7</span></label>
                            <label class="cursor-pointer select-none"><input type="checkbox" value="0" class="peer sr-only"><span class="px-2 py-1 rounded-lg text-[11px] font-semibold bg-white border border-nude-300 text-forest-900 peer-checked:bg-forest-900 peer-checked:text-nude-50 peer-checked:border-forest-900 inline-block transition">CN</span></label>
                        </div>

                        <div class="grid grid-cols-1 sm:grid-cols-2 gap-2">
                            <input type="text" id="noteClassSlot" placeholder="Ca & Giờ (VD: Ca 3 • 12h00 - 14h00)"
                                class="w-full px-3 py-2 rounded-xl bg-white border border-nude-300 text-forest-950 text-xs focus:outline-none focus:ring-2 focus:ring-forest-700 transition">
                            <input type="text" id="noteInstructor" placeholder="Môn / GV (VD: IT - Cô Vân)"
                                class="w-full px-3 py-2 rounded-xl bg-white border border-nude-300 text-forest-950 text-xs focus:outline-none focus:ring-2 focus:ring-forest-700 transition">
                        </div>
                    </div>

                    <!-- Trạng thái & Ghim yêu thích -->
                    <div class="grid grid-cols-1 sm:grid-cols-2 gap-3 items-end">
                        <div>
                            <label for="noteStatus" class="block text-xs font-semibold uppercase tracking-wider text-forest-900 mb-1.5">
                                Trạng thái
                            </label>
                            <select id="noteStatus"
                                class="w-full px-3 py-2.5 rounded-xl bg-white border border-nude-300 text-forest-950 text-sm focus:outline-none focus:ring-2 focus:ring-forest-700 transition">
                                <option value="todo" selected>⏳ Chưa làm</option>
                                <option value="in_progress">⚡ Đang làm</option>
                                <option value="done">✅ Hoàn thành</option>
                            </select>
                        </div>

                        <label class="flex items-center gap-2.5 px-3.5 py-2.5 rounded-xl bg-white border border-nude-300 cursor-pointer hover:border-nude-400 transition select-none">
                            <input type="checkbox" id="notePinned" class="w-4 h-4 accent-forest-900 rounded cursor-pointer">
                            <span class="text-sm font-medium text-forest-900 flex items-center gap-1.5">
                                <i class="ph-fill ph-push-pin text-nude-500"></i>
                                Ghim yêu thích
                            </span>
                        </label>
                    </div>

                    <!-- Nút Submit -->
                    <div class="pt-2 flex items-center gap-2">
                        <button type="submit"
                            class="flex-1 bg-forest-900 hover:bg-forest-800 active:scale-[0.99] text-nude-50 font-semibold py-3 px-4 rounded-xl shadow-md transition flex items-center justify-center gap-2 cursor-pointer">
                            <i class="ph-bold ph-plus text-lg text-nude-400"></i>
                            <span>Lưu Ghi Chú</span>
                        </button>
                        <button type="button" onclick="resetCreateForm()" title="Làm mới form"
                            class="p-3 rounded-xl bg-nude-200 hover:bg-nude-300 text-forest-900 transition cursor-pointer">
                            <i class="ph ph-arrow-counter-clockwise text-lg"></i>
                        </button>
                    </div>
                </form>
            </aside>

            <!-- CỘT PHẢI: BỘ LỌC, SẮP XẾP & DANH SÁCH THẺ GHI CHÚ (8 CỘT TRÊN DESKTOP) -->
            <section class="lg:col-span-8 space-y-5">

                <!-- Thanh Thống kê nhanh 4 chỉ số -->
                <div class="grid grid-cols-2 sm:grid-cols-4 gap-3">
                    <div class="bg-nude-50 border border-nude-300 rounded-2xl p-3.5 shadow-card flex items-center justify-between">
                        <div>
                            <p class="text-xs font-medium text-nude-800">Tổng ghi chú</p>
                            <p id="statTotal" class="font-display font-bold text-xl sm:text-2xl text-forest-900 mt-0.5">0</p>
                        </div>
                        <div class="w-9 h-9 rounded-xl bg-forest-100 text-forest-800 flex items-center justify-center shrink-0">
                            <i class="ph-fill ph-files text-lg"></i>
                        </div>
                    </div>

                    <div class="bg-nude-50 border border-nude-300 rounded-2xl p-3.5 shadow-card flex items-center justify-between">
                        <div>
                            <p class="text-xs font-medium text-nude-800">Đang làm</p>
                            <p id="statInProgress" class="font-display font-bold text-xl sm:text-2xl text-amber-600 mt-0.5">0</p>
                        </div>
                        <div class="w-9 h-9 rounded-xl bg-amber-100 text-amber-700 flex items-center justify-center shrink-0">
                            <i class="ph-fill ph-lightning text-lg"></i>
                        </div>
                    </div>

                    <div class="bg-nude-50 border border-nude-300 rounded-2xl p-3.5 shadow-card flex items-center justify-between">
                        <div>
                            <p class="text-xs font-medium text-nude-800">Đã ghim</p>
                            <p id="statPinned" class="font-display font-bold text-xl sm:text-2xl text-nude-500 mt-0.5">0</p>
                        </div>
                        <div class="w-9 h-9 rounded-xl bg-nude-200 text-nude-800 flex items-center justify-center shrink-0">
                            <i class="ph-fill ph-push-pin text-lg"></i>
                        </div>
                    </div>

                    <div class="bg-nude-50 border border-nude-300 rounded-2xl p-3.5 shadow-card flex items-center justify-between">
                        <div>
                            <p class="text-xs font-medium text-nude-800">Hoàn thành</p>
                            <p id="statCompleted" class="font-display font-bold text-xl sm:text-2xl text-forest-700 mt-0.5">0</p>
                        </div>
                        <div class="w-9 h-9 rounded-xl bg-forest-100 text-forest-700 flex items-center justify-center shrink-0">
                            <i class="ph-fill ph-check-circle text-lg"></i>
                        </div>
                    </div>
                </div>

                <!-- Thanh Công Cụ Tìm Kiếm, Lọc Đa Chiều & Sắp Xếp -->
                <div class="bg-nude-50 border border-nude-300 rounded-2xl p-4 shadow-card space-y-3.5">
                    <div class="flex flex-col sm:flex-row gap-3">
                        <!-- Ô tìm kiếm -->
                        <div class="relative flex-1">
                            <i class="ph ph-magnifying-glass absolute left-3.5 top-1/2 -translate-y-1/2 text-nude-800 text-lg"></i>
                            <input type="text" id="searchInput" oninput="renderNotes()" placeholder="Tìm tiêu đề, nội dung, ngày dd/mm/yyyy..."
                                class="w-full pl-10 pr-4 py-2.5 rounded-xl bg-white border border-nude-300 text-forest-950 text-sm focus:outline-none focus:ring-2 focus:ring-forest-700 transition">
                        </div>

                        <!-- Bộ chọn Sắp xếp -->
                        <div class="flex items-center gap-2 shrink-0">
                            <label for="sortSelect" class="text-xs font-semibold text-forest-900 whitespace-nowrap flex items-center gap-1">
                                <i class="ph ph-arrows-down-up text-base text-forest-700"></i>
                                Sắp xếp:
                            </label>
                            <select id="sortSelect" onchange="handleSortChange()"
                                class="px-3 py-2.5 rounded-xl bg-white border border-nude-300 text-forest-950 text-sm font-medium focus:outline-none focus:ring-2 focus:ring-forest-700 transition cursor-pointer">
                                <option value="manual">✋ Thứ tự tùy chỉnh (Kéo thả)</option>
                                <option value="due_asc">⏰ Ngày đến hạn (Gần nhất trước)</option>
                                <option value="due_desc">⏰ Ngày đến hạn (Xa nhất trước)</option>
                                <option value="created_desc">🆕 Ngày tạo (Mới nhất)</option>
                                <option value="updated_desc">🔄 Ngày cập nhật (Gần nhất)</option>
                                <option value="priority_desc">🔥 Mức độ ưu tiên (Cao → Thấp)</option>
                                <option value="priority_asc">🌿 Mức độ ưu tiên (Thấp → Cao)</option>
                            </select>
                        </div>
                    </div>

                    <!-- Bộ lọc kết hợp: Trạng thái, Danh mục, Ưu tiên, Ghim -->
                    <div class="grid grid-cols-2 sm:grid-cols-4 gap-2.5 pt-2 border-t border-nude-200">
                        <div>
                            <select id="filterStatus" onchange="renderNotes()" aria-label="Lọc theo trạng thái"
                                class="w-full px-3 py-2 rounded-xl bg-white border border-nude-300 text-forest-900 text-xs font-semibold focus:outline-none focus:ring-2 focus:ring-forest-700 cursor-pointer">
                                <option value="all">📋 Tất cả trạng thái</option>
                                <option value="todo">⏳ Chưa làm</option>
                                <option value="in_progress">⚡ Đang làm</option>
                                <option value="done">✅ Hoàn thành</option>
                            </select>
                        </div>

                        <div>
                            <select id="filterCategory" onchange="renderNotes()" aria-label="Lọc theo danh mục"
                                class="w-full px-3 py-2 rounded-xl bg-white border border-nude-300 text-forest-900 text-xs font-semibold focus:outline-none focus:ring-2 focus:ring-forest-700 cursor-pointer">
                                <option value="all">🗂️ Tất cả danh mục</option>
                                <option value="Học tập">📚 Học tập</option>
                                <option value="Lịch học">🎓 Lịch học</option>
                                <option value="Công việc">💼 Công việc</option>
                                <option value="Cá nhân">🌿 Cá nhân</option>
                                <option value="Sức khỏe">🧘 Sức khỏe</option>
                                <option value="Khác">📌 Khác</option>
                            </select>
                        </div>

                        <div>
                            <select id="filterPriority" onchange="renderNotes()" aria-label="Lọc theo mức độ ưu tiên"
                                class="w-full px-3 py-2 rounded-xl bg-white border border-nude-300 text-forest-900 text-xs font-semibold focus:outline-none focus:ring-2 focus:ring-forest-700 cursor-pointer">
                                <option value="all">🎯 Tất cả ưu tiên</option>
                                <option value="high">🔴 Ưu tiên Cao</option>
                                <option value="normal">🟡 Ưu tiên Bình thường</option>
                                <option value="low">🟢 Ưu tiên Thấp</option>
                            </select>
                        </div>

                        <button type="button" id="btnFilterPinned" onclick="togglePinnedFilter()"
                            class="px-3 py-2 rounded-xl bg-nude-200 hover:bg-nude-300 text-forest-900 text-xs font-semibold transition flex items-center justify-center gap-1.5 cursor-pointer">
                            <i class="ph-fill ph-push-pin text-nude-500"></i>
                            <span>Chỉ xem đã ghim</span>
                        </button>
                    </div>

                    <div class="flex flex-wrap items-center justify-between gap-2 text-[11px] text-nude-800">
                        <span class="flex items-center gap-1">
                            <i class="ph-fill ph-push-pin text-nude-500"></i>
                            Ghi chú <strong>Ghim yêu thích</strong> luôn ưu tiên hiển thị lên đầu danh sách
                        </span>
                        <span id="manualSortHint" class="text-forest-700 font-medium flex items-center gap-1">
                            <i class="ph ph-dots-six-vertical text-sm"></i>
                            Kéo thả thẻ hoặc bấm mũi tên để đổi thứ tự
                        </span>
                    </div>
                </div>

                <!-- Danh sách các thẻ ghi chú -->
                <div id="notesContainer" class="grid grid-cols-1 md:grid-cols-2 gap-4">
                    <!-- Các thẻ ghi chú sẽ được render bằng JavaScript tại đây -->
                </div>

                <!-- Trạng thái trống (Empty State) -->
                <div id="emptyState" class="hidden bg-nude-50 border border-dashed border-nude-400 rounded-2xl p-10 text-center">
                    <div class="w-16 h-16 rounded-2xl bg-nude-200 text-forest-800 flex items-center justify-center mx-auto mb-3">
                        <i class="ph ph-note-blank text-3xl"></i>
                    </div>
                    <h3 class="font-display font-bold text-lg text-forest-900">Không tìm thấy ghi chú nào</h3>
                    <p class="text-sm text-nude-800 mt-1 max-w-md mx-auto">
                        Hãy thêm ghi chú mới ở bảng bên cạnh hoặc đặt lại bộ lọc tìm kiếm hiện tại.
                    </p>
                    <button type="button" onclick="resetAllFilters()"
                        class="mt-4 px-4 py-2 rounded-xl bg-forest-900 hover:bg-forest-800 text-nude-50 text-xs font-semibold transition cursor-pointer">
                        Đặt lại bộ lọc
                    </button>
                </div>

            </section>
        </div>
    </main>

    <!-- ==================== MODAL CHỈNH SỬA GHI CHÚ ==================== -->
    <div id="editModal" class="fixed inset-0 z-50 hidden bg-forest-950/60 backdrop-blur-xs flex items-center justify-center p-4 overflow-y-auto">
        <div class="bg-nude-50 border border-nude-300 rounded-2xl max-w-lg w-full p-5 sm:p-6 shadow-2xl transform transition-all my-auto">
            <div class="flex items-center justify-between pb-3 mb-4 border-b border-nude-200">
                <div>
                    <h3 class="font-display font-bold text-lg text-forest-900 flex items-center gap-2">
                        <i class="ph-fill ph-pencil-simple-line text-forest-700 text-xl"></i>
                        Chỉnh Sửa Ghi Chú
                    </h3>
                    <p id="editMetaTimestamps" class="text-[11px] text-nude-800 mt-0.5"></p>
                </div>
                <button onclick="closeEditModal()" class="p-1.5 rounded-lg hover:bg-nude-200 text-forest-900 transition cursor-pointer">
                    <i class="ph ph-x text-xl"></i>
                </button>
            </div>

            <form id="editNoteForm" onsubmit="handleUpdateNote(event)" class="space-y-4">
                <input type="hidden" id="editNoteId">

                <div>
                    <label for="editNoteTitle" class="block text-xs font-semibold uppercase tracking-wider text-forest-900 mb-1.5">
                        Tiêu đề ghi chú <span class="text-red-600">*</span>
                    </label>
                    <input type="text" id="editNoteTitle" required
                        class="w-full px-3.5 py-2.5 rounded-xl bg-white border border-nude-300 text-forest-950 text-sm focus:outline-none focus:ring-2 focus:ring-forest-700">
                </div>

                <div>
                    <label for="editNoteContent" class="block text-xs font-semibold uppercase tracking-wider text-forest-900 mb-1.5">
                        Nội dung chi tiết
                    </label>
                    <textarea id="editNoteContent" rows="4"
                        class="w-full px-3.5 py-2.5 rounded-xl bg-white border border-nude-300 text-forest-950 text-sm focus:outline-none focus:ring-2 focus:ring-forest-700 resize-y"></textarea>
                </div>

                <div class="grid grid-cols-1 sm:grid-cols-2 gap-3">
                    <div>
                        <label for="editNoteDueDate" class="block text-xs font-semibold uppercase tracking-wider text-forest-900 mb-1.5">
                            Ngày đến hạn <span class="text-red-600">*</span>
                        </label>
                        <input type="date" id="editNoteDueDate" required
                            class="w-full px-3 py-2.5 rounded-xl bg-white border border-nude-300 text-forest-950 text-sm focus:outline-none focus:ring-2 focus:ring-forest-700">
                        <p id="editFormattedDatePreview" class="text-[11px] text-forest-700 font-medium mt-1">Deadline: --/--/----</p>
                    </div>

                    <div>
                        <label for="editNoteReminderTime" class="block text-xs font-semibold uppercase tracking-wider text-forest-900 mb-1.5">
                            Giờ nhắc nhở
                        </label>
                        <input type="time" id="editNoteReminderTime"
                            class="w-full px-3 py-2.5 rounded-xl bg-white border border-nude-300 text-forest-950 text-sm focus:outline-none focus:ring-2 focus:ring-forest-700">
                    </div>
                </div>

                <div class="grid grid-cols-1 sm:grid-cols-2 gap-3">
                    <div>
                        <label for="editNoteCategory" class="block text-xs font-semibold uppercase tracking-wider text-forest-900 mb-1.5">
                            Danh mục
                        </label>
                        <select id="editNoteCategory"
                            class="w-full px-3 py-2.5 rounded-xl bg-white border border-nude-300 text-forest-950 text-sm focus:outline-none focus:ring-2 focus:ring-forest-700">
                            <option value="Học tập">📚 Học tập</option>
                            <option value="Lịch học">🎓 Lịch học</option>
                            <option value="Công việc">💼 Công việc</option>
                            <option value="Cá nhân">🌿 Cá nhân</option>
                            <option value="Sức khỏe">🧘 Sức khỏe</option>
                            <option value="Khác">📌 Khác</option>
                        </select>
                    </div>

                    <div>
                        <label for="editNotePriority" class="block text-xs font-semibold uppercase tracking-wider text-forest-900 mb-1.5">
                            Mức độ ưu tiên
                        </label>
                        <select id="editNotePriority"
                            class="w-full px-3 py-2.5 rounded-xl bg-white border border-nude-300 text-forest-950 text-sm focus:outline-none focus:ring-2 focus:ring-forest-700">
                            <option value="low">🟢 Thấp</option>
                            <option value="normal">🟡 Bình thường</option>
                            <option value="high">🔴 Cao</option>
                        </select>
                    </div>
                </div>

                <!-- Chỉnh sửa Lịch học định kỳ -->
                <div class="bg-nude-100/80 border border-nude-300 rounded-xl p-3 space-y-2.5">
                    <span class="block text-xs font-semibold uppercase tracking-wider text-forest-900">
                        🎓 Lịch học định kỳ (Thứ 2 - CN)
                    </span>
                    <div class="flex flex-wrap gap-1.5" id="editDaysPicker">
                        <label class="cursor-pointer select-none"><input type="checkbox" value="1" class="peer sr-only"><span class="px-2 py-1 rounded-lg text-[11px] font-semibold bg-white border border-nude-300 text-forest-900 peer-checked:bg-forest-900 peer-checked:text-nude-50 peer-checked:border-forest-900 inline-block transition">T2</span></label>
                        <label class="cursor-pointer select-none"><input type="checkbox" value="2" class="peer sr-only"><span class="px-2 py-1 rounded-lg text-[11px] font-semibold bg-white border border-nude-300 text-forest-900 peer-checked:bg-forest-900 peer-checked:text-nude-50 peer-checked:border-forest-900 inline-block transition">T3</span></label>
                        <label class="cursor-pointer select-none"><input type="checkbox" value="3" class="peer sr-only"><span class="px-2 py-1 rounded-lg text-[11px] font-semibold bg-white border border-nude-300 text-forest-900 peer-checked:bg-forest-900 peer-checked:text-nude-50 peer-checked:border-forest-900 inline-block transition">T4</span></label>
                        <label class="cursor-pointer select-none"><input type="checkbox" value="4" class="peer sr-only"><span class="px-2 py-1 rounded-lg text-[11px] font-semibold bg-white border border-nude-300 text-forest-900 peer-checked:bg-forest-900 peer-checked:text-nude-50 peer-checked:border-forest-900 inline-block transition">T5</span></label>
                        <label class="cursor-pointer select-none"><input type="checkbox" value="5" class="peer sr-only"><span class="px-2 py-1 rounded-lg text-[11px] font-semibold bg-white border border-nude-300 text-forest-900 peer-checked:bg-forest-900 peer-checked:text-nude-50 peer-checked:border-forest-900 inline-block transition">T6</span></label>
                        <label class="cursor-pointer select-none"><input type="checkbox" value="6" class="peer sr-only"><span class="px-2 py-1 rounded-lg text-[11px] font-semibold bg-white border border-nude-300 text-forest-900 peer-checked:bg-forest-900 peer-checked:text-nude-50 peer-checked:border-forest-900 inline-block transition">T7</span></label>
                        <label class="cursor-pointer select-none"><input type="checkbox" value="0" class="peer sr-only"><span class="px-2 py-1 rounded-lg text-[11px] font-semibold bg-white border border-nude-300 text-forest-900 peer-checked:bg-forest-900 peer-checked:text-nude-50 peer-checked:border-forest-900 inline-block transition">CN</span></label>
                    </div>
                    <div class="grid grid-cols-1 sm:grid-cols-2 gap-2">
                        <input type="text" id="editNoteClassSlot" placeholder="Ca & Giờ (VD: Ca 3 • 12h00 - 14h00)"
                            class="w-full px-3 py-2 rounded-xl bg-white border border-nude-300 text-forest-950 text-xs focus:outline-none focus:ring-2 focus:ring-forest-700">
                        <input type="text" id="editNoteInstructor" placeholder="Môn / GV (VD: IT - Cô Vân)"
                            class="w-full px-3 py-2 rounded-xl bg-white border border-nude-300 text-forest-950 text-xs focus:outline-none focus:ring-2 focus:ring-forest-700">
                    </div>
                </div>

                <div class="grid grid-cols-1 sm:grid-cols-2 gap-3 items-end">
        // Lấy chuỗi ngày hôm nay định dạng YYYY-MM-DD
        function getTodayISO() {
            const now = new Date();
            const year = now.getFullYear();
            const month = String(now.getMonth() + 1).padStart(2, '0');
            const day = String(now.getDate()).padStart(2, '0');
            return `${year}-${month}-${day}`;
        }

        // Lấy chuỗi ngày theo ngày/tháng trong năm hiện tại định dạng YYYY-MM-DD
        function getDateISOInCurrentYear(dayOfMonth, monthNumber = null) {
            const now = new Date();
            const year = now.getFullYear();
            const monthIndex = monthNumber !== null ? (monthNumber - 1) : now.getMonth();
            const candidate = new Date(year, monthIndex, dayOfMonth);
            const y = candidate.getFullYear();
            const m = String(candidate.getMonth() + 1).padStart(2, '0');
            const d = String(candidate.getDate()).padStart(2, '0');
            return `${y}-${m}-${d}`;
        }

        // Lấy chuỗi ngày mai định dạng YYYY-MM-DD
        function getTomorrowISO() {
        // Dữ liệu mẫu khởi tạo lần đầu (bao gồm các lịch học định kỳ và các mục ghi chú chi tiêu của bạn)
        function getDefaultSeedNotes() {
            const today = getTodayISO();
            const tomorrow = getTomorrowISO();
            const now = Date.now();
            const monToSat = [1, 2, 3, 4, 5, 6]; // Thứ 2 đến Thứ 7
            const monToFri = [1, 2, 3, 4, 5];    // Thứ 2 đến Thứ 6

            return [
                normalizeNoteSchema({
                    id: 'note_' + now + '_class_it',
                    title: 'Học Ca 3 (12h00 - 14h00) môn IT – Cô Vân',
                    content: 'Lịch học định kỳ từ Thứ 2 đến Thứ 7 • Ca 3 (12h00 - 14h00). Chuẩn bị laptop, tài liệu thực hành và bài tập môn IT.',
                    createdAt: now - 3600000,
                    updatedAt: now - 600000,
                    dueDate: getNextMatchingDateISO(monToSat),
                    reminderTime: '12:00',
                    category: 'Lịch học',
                    priority: 'high',
                    status: 'in_progress',
                    isPinned: true,
                    repeatDays: monToSat,
                    classSlot: 'Ca 3 • 12h00 - 14h00',
                    instructor: 'Môn IT • Cô Vân',
                    notified: false,
                    order: 0
                }, 0),
                normalizeNoteSchema({
                    id: 'note_' + now + '_class_ita',
                    title: 'Học Chiều Ca 4 môn ITA – Cô Diệu',
                    content: 'Lịch học định kỳ từ Thứ 2 đến Thứ 7 • Buổi chiều Ca 4. Chuẩn bị giáo trình và bài tập môn ITA.',
                    createdAt: now - 3500000,
                    updatedAt: now - 500000,
                    dueDate: getNextMatchingDateISO(monToSat),
                    reminderTime: '14:30',
                    category: 'Lịch học',
                    priority: 'high',
                    status: 'in_progress',
                    isPinned: true,
                    repeatDays: monToSat,
                    classSlot: 'Chiều Ca 4',
                    instructor: 'Môn ITA • Cô Diệu',
                    notified: false,
                    order: 1
                }, 1),
                normalizeNoteSchema({
                    id: 'note_' + now + '_class_eng',
                    title: 'Học Tiếng Anh (19h00 - 21h00) – Thầy Hùng',
                    content: 'Lịch học định kỳ từ Thứ 2 đến Thứ 6 • Khung giờ tối 19h00 - 21h00. Luyện giao tiếp, từ vựng và ngữ pháp Tiếng Anh.',
                    createdAt: now - 3400000,
                    updatedAt: now - 400000,
                    dueDate: getNextMatchingDateISO(monToFri),
                    reminderTime: '19:00',
                    category: 'Lịch học',
                    priority: 'high',
                    status: 'in_progress',
                    isPinned: true,
                    repeatDays: monToFri,
                    classSlot: 'Tối • 19h00 - 21h00',
                    instructor: 'Tiếng Anh • Thầy Hùng',
                    notified: false,
                    order: 2
                }, 2),
                normalizeNoteSchema({
                    id: 'note_' + now + '_exp_repair',
                    title: 'Sửa xe – 11tr (Ngày 24)',
                    content: 'Khoản mục sửa xe: 11.000.000đ (11tr), ghi nhận ngày 24.',
                    createdAt: now - 3300000,
                    updatedAt: now - 350000,
                    dueDate: getDateISOInCurrentYear(24),
                    reminderTime: '',
                    category: 'Cá nhân',
                    priority: 'high',
                    status: 'done',
                    isPinned: false,
                    notified: false,
                    order: 3
                }, 3),
                normalizeNoteSchema({
                    id: 'note_' + now + '_exp_rental',
                    title: 'Thuê xe – 1tr9 (Ngày 23/11)',
                    content: 'Khoản mục thuê xe: 1.900.000đ (1tr9), ngày 23/11.',
                    createdAt: now - 3200000,
                    updatedAt: now - 340000,
                    dueDate: getDateISOInCurrentYear(23, 11),
                    reminderTime: '',
                    category: 'Cá nhân',
                    priority: 'normal',
                    status: 'done',
                    isPinned: false,
                    notified: false,
                    order: 4
                }, 4),
                normalizeNoteSchema({
                    id: 'note_' + now + '_exp_gas_26',
                    title: 'Xăng – 1.500 (Ngày 26)',
                    content: 'Ghi chú đổ xăng: 1.500, ngày 26.',
                    createdAt: now - 3100000,
                    updatedAt: now - 330000,
                    dueDate: getDateISOInCurrentYear(26),
                    reminderTime: '',
                    category: 'Cá nhân',
                    priority: 'normal',
                    status: 'done',
                    isPinned: false,
                    notified: false,
                    order: 5
                }, 5),
                normalizeNoteSchema({
                    id: 'note_' + now + '_exp_tpbank_25',
                    title: 'TPBank – 7.000 (Ngày 25)',
                    content: 'Giao dịch / ghi chú TPBank: 7.000, ngày 25.',
                    createdAt: now - 3000000,
                    updatedAt: now - 320000,
                    dueDate: getDateISOInCurrentYear(25),
                    reminderTime: '',
                    category: 'Cá nhân',
                    priority: 'normal',
                    status: 'done',
                    isPinned: false,
                    notified: false,
                    order: 6
                }, 6),
                normalizeNoteSchema({
                    id: 'note_' + now + '_exp_gas_8',
                    title: 'Đổ xăng – 1tr500 (Ngày 8)',
                    content: 'Đổ xăng: 1.500.000đ (1tr500), ngày 8.',
                    createdAt: now - 2900000,
                    updatedAt: now - 310000,
                    dueDate: getDateISOInCurrentYear(8),
                    reminderTime: '',
                    category: 'Cá nhân',
                    priority: 'normal',
                    status: 'done',
                    isPinned: false,
                    notified: false,
                    order: 7
                }, 7),
                normalizeNoteSchema({
                    id: 'note_' + now + '_exp_laundry',
                    title: 'Giặt chăn – 500',
                    content: 'Chi phí giặt chăn: 500.',
                    createdAt: now - 2800000,
                    updatedAt: now - 300000,
                    dueDate: today,
                    reminderTime: '',
                    category: 'Cá nhân',
                    priority: 'low',
                    status: 'done',
                    isPinned: false,
                    notified: false,
                    order: 8
                }, 8),
                normalizeNoteSchema({
                    id: 'note_' + now + '_exp_supermarket',
                    title: 'Ăn uống đi siêu thị ăn – 5tr',
                    content: 'Chi tiêu ăn uống, đi siêu thị ăn: 5.000.000đ (5tr).',
                    createdAt: now - 2700000,
                    updatedAt: now - 290000,
                    dueDate: today,
                    reminderTime: '',
                    category: 'Cá nhân',
                    priority: 'high',
                    status: 'done',
                    isPinned: false,
                    notified: false,
                    order: 9
                }, 9),
                normalizeNoteSchema({
                    id: 'note_' + now + '_1',
                    title: 'Hoàn thiện Backend API với FastAPI / Django',
                    content: 'Kiểm tra các endpoint CRUD, cấu hình CORS và kết nối cơ sở dữ liệu PostgreSQL cho dự án cá nhân.',
                    createdAt: now - 7200000,
                    updatedAt: now - 1800000,
                    dueDate: today,
                    reminderTime: '16:30',
                    category: 'Công việc',
                    priority: 'high',
                    status: 'in_progress',
                    isPinned: false,
                    notified: false,
                    order: 10
                }, 10),
                normalizeNoteSchema({
                    id: 'note_' + now + '_2',
                    title: 'Đọc sách & Tổng hợp kiến thức System Design',
                    content: 'Đọc chương 3 về Caching (Redis) và tối ưu hóa truy vấn SQL. Ghi chú lại các mẫu kiến trúc quan trọng.',
                    createdAt: now - 86400000,
                    updatedAt: now - 43200000,
                    dueDate: tomorrow,
                    reminderTime: '20:00',
                    category: 'Học tập',
                    priority: 'normal',
                    status: 'todo',
                    isPinned: false,
                    notified: false,
                    order: 11
                }, 11),
                normalizeNoteSchema({
                    id: 'note_' + now + '_3',
                    title: 'Chạy bộ 5km & Uống đủ 2 lít nước mỗi ngày',
                    content: 'Duy trì thói quen vận động buổi chiều để tăng cường sức khỏe và sự tập trung.',
                    createdAt: now - 172800000,
                    updatedAt: now - 3600000,
                    dueDate: today,
                    reminderTime: '',
                    category: 'Sức khỏe',
                    priority: 'low',
                    status: 'done',
                    isPinned: false,
                    notified: false,
                    order: 12
                }, 12)
            ];
        }

        // Tải dữ liệu từ LocalStorage (hỗ trợ di trú tự động & tự thêm các mục mới nếu chưa có)
        function loadFromStorage() {
            try {
                const rawData = localStorage.getItem(STORAGE_KEY);
                if (rawData !== null) {
                    const parsed = JSON.parse(rawData) || [];
                    notes = parsed.map((item, idx) => normalizeNoteSchema(item, idx));
                    const seeds = getDefaultSeedNotes();
                    let addedCount = 0;

                    // Kiểm tra và bổ sung các lịch học & mục ghi chú mới yêu cầu nếu chưa có trong LocalStorage hiện tại
                    const requiredSchedules = [
                        { check: n => n.title.includes('Cô Vân') && n.title.includes('Ca 3'), seed: seeds[0] },
                        { check: n => n.title.includes('Cô Diệu') && n.title.includes('ITA'), seed: seeds[1] },
                        { check: n => n.title.includes('Thầy Hùng') && n.title.toLowerCase().includes('tiếng anh'), seed: seeds[2] },
                        { check: n => n.title.toLowerCase().includes('sửa xe') && n.title.includes('11tr'), seed: seeds[3] },
                        { check: n => n.title.toLowerCase().includes('thuê xe') && n.title.includes('1tr9'), seed: seeds[4] },
                        { check: n => n.title.toLowerCase().includes('xăng') && n.title.includes('1.500') && n.title.includes('26'), seed: seeds[5] },
                        { check: n => n.title.toLowerCase().includes('tpbank') && n.title.includes('7.000'), seed: seeds[6] },
                        { check: n => n.title.toLowerCase().includes('đổ xăng') && n.title.includes('1tr500'), seed: seeds[7] },
                        { check: n => n.title.toLowerCase().includes('giặt chăn') && n.title.includes('500'), seed: seeds[8] },
                        { check: n => n.title.toLowerCase().includes('siêu thị ăn') && n.title.includes('5tr'), seed: seeds[9] }
                    ];

                    requiredSchedules.reverse().forEach(item => {
                        if (!notes.some(item.check)) {
                            notes.forEach(n => n.order = (n.order ?? 0) + 1);
                            notes.unshift(item.seed);
                            addedCount++;
                        }
                    });

                    if (addedCount > 0) {
                        saveToStorage();
                    }
                    return;
                }

                const legacyData = localStorage.getItem(LEGACY_STORAGE_KEY);
                if (legacyData !== null) {
                    const parsedLegacy = JSON.parse(legacyData) || [];
                    notes = parsedLegacy.map((item, idx) => normalizeNoteSchema(item, idx));
                    saveToStorage();
                    return;
                }

                notes = getDefaultSeedNotes();
                saveToStorage();
            } catch (error) {
                console.error('Lỗi đọc LocalStorage:', error);
                notes = [];
            }
        }

        // Lưu toàn bộ dữ liệu xuống LocalStorage
        function saveToStorage() {
            try {
                localStorage.setItem(STORAGE_KEY, JSON.stringify(notes));
            } catch (error) {
                console.error('Lỗi lưu LocalStorage:', error);
                showToast('Không thể lưu dữ liệu vào trình duyệt (bộ nhớ đầy).', 'error');
            }
        }

        /**
         * ====================================================================
         * 2. ÂM THANH NHẮC NHỞ (WEB AUDIO API) & WEB NOTIFICATION
         * ====================================================================
         */
        function playChimeSound() {
            try {
                const AudioCtx = window.AudioContext || window.webkitAudioContext;
                if (!AudioCtx) return;
                const ctx = new AudioCtx();

                // Tạo 2 nốt chuông êm dịu (C5 -> E5)
                const notesFreq = [523.25, 659.25];
                notesFreq.forEach((freq, idx) => {
                    const osc = ctx.createOscillator();
                    const gain = ctx.createGain();
                    osc.type = 'sine';
                    osc.frequency.setValueAtTime(freq, ctx.currentTime + idx * 0.18);

                    gain.gain.setValueAtTime(0.001, ctx.currentTime + idx * 0.18);
                    gain.gain.exponentialRampToValueAtTime(0.18, ctx.currentTime + idx * 0.18 + 0.04);
                    gain.gain.exponentialRampToValueAtTime(0.0001, ctx.currentTime + idx * 0.18 + 0.7);

                    osc.connect(gain);
                    gain.connect(ctx.destination);

                    osc.start(ctx.currentTime + idx * 0.18);
                    osc.stop(ctx.currentTime + idx * 0.18 + 0.75);
                });
            } catch (e) {
                console.warn('Trình duyệt chặn tự động phát âm thanh khi chưa tương tác:', e);
            }
        }

        function updateNotifButtonUI() {
            const statusText = document.getElementById('notifStatusText');
            const notifIcon = document.getElementById('notifIcon');
            if (!('Notification' in window)) {
                statusText.textContent = 'Không hỗ trợ';
                return;
            }
            if (Notification.permission === 'granted') {
                statusText.textContent = 'Đã bật thông báo';
                notifIcon.className = 'ph-fill ph-bell-ringing text-base text-nude-400';
            } else if (Notification.permission === 'denied') {
                statusText.textContent = 'Đã chặn thông báo';
                notifIcon.className = 'ph ph-bell-slash text-base text-red-400';
            } else {
                statusText.textContent = 'Bật thông báo';
                notifIcon.className = 'ph ph-bell text-base text-nude-400';
            }
        }

        function requestBrowserNotification() {
            if (!('Notification' in window)) {
                showToast('Trình duyệt của bạn không hỗ trợ Web Notification.', 'warning');
                return;
            }
            Notification.requestPermission().then(permission => {
                updateNotifButtonUI();
                if (permission === 'granted') {
                    showToast('Đã bật thông báo nhắc nhở trên trình duyệt!', 'success');
                    playChimeSound();
                } else {
                    showToast('Bạn chưa cấp quyền thông báo trình duyệt.', 'warning');
                }
            // Kiểm tra từng ghi chú có đặt giờ nhắc nhở (hỗ trợ cả lịch học lặp lại theo thứ T2-CN)
            const todayISO = getTodayISO();
            const todayDow = now.getDay();
            const currentHM = `${String(now.getHours()).padStart(2, '0')}:${String(now.getMinutes()).padStart(2, '0')}`;
            let hasNewTrigger = false;

            notes.forEach(note => {
                if (note.status !== 'done' && note.reminderTime) {
                    const isRecurringToday = Array.isArray(note.repeatDays) && note.repeatDays.includes(todayDow);
                    const isExactDateToday = note.dueDate === todayISO;

                    if (isRecurringToday) {
                        // Tự động cập nhật ngày đến hạn sang ngày học hiện tại nếu sang ngày mới
                        if (note.dueDate < todayISO) {
                            note.dueDate = todayISO;
                            note.notified = false;
                        }
                        if (note.lastNotifiedDate !== todayISO && note.reminderTime <= currentHM) {
                            note.notified = true;
                            note.lastNotifiedDate = todayISO;
                            hasNewTrigger = true;
                            triggerReminderAlert(note);
                        }
                    } else if (isExactDateToday && !note.notified && note.reminderTime <= currentHM) {
                        note.notified = true;
                        note.lastNotifiedDate = todayISO;
                        hasNewTrigger = true;
                        triggerReminderAlert(note);
                    }
                }
            });

            if (hasNewTrigger) {
                saveToStorage();
                renderNotes();
                renderUpcomingReminders();
            }
        }

        function triggerReminderAlert(note) {
            activeAlarmNoteId = note.id;
            const banner = document.getElementById('activeAlarmBanner');
            const titleEl = document.getElementById('activeAlarmTitle');
            titleEl.textContent = `${note.title} (Hạn: ${formatDateVN(note.dueDate)} lúc ${note.reminderTime})`;
            banner.classList.remove('hidden');

            playChimeSound();
            showToast(`⏰ Nhắc nhở: "${note.title}" (${note.reminderTime})`, 'reminder');

            if ('Notification' in window && Notification.permission === 'granted') {
                new Notification('EmeraldNote - Nhắc nhở công việc', {
                    body: `${note.title}\nHạn chót: ${formatDateVN(note.dueDate)} lúc ${note.reminderTime}`,
                });
            }
        }

        function dismissActiveAlarm() {
            document.getElementById('activeAlarmBanner').classList.add('hidden');
            activeAlarmNoteId = null;
        }

        function completeActiveAlarm() {
            if (activeAlarmNoteId) {
                changeNoteStatus(activeAlarmNoteId, 'done');
            }
            dismissActiveAlarm();
        }

        /**
         * ====================================================================
         * 4. CÁC THAO TÁC CRUD (THÊM, SỬA, XÓA, ĐỔI TRẠNG THÁI, GHIM YÊU THÍCH)
         * ====================================================================
         */
        function handleCreateNote(e) {
            e.preventDefault();

            const title = document.getElementById('noteTitle').value.trim();
            const content = document.getElementById('noteContent').value.trim();
            const repeatDays = getSelectedDaysFromPicker('createDaysPicker');
            const dueDate = document.getElementById('noteDueDate').value || getNextMatchingDateISO(repeatDays);
            const reminderTime = document.getElementById('noteReminderTime').value;
            const category = document.getElementById('noteCategory').value;
            const priority = document.getElementById('notePriority').value;
            const status = document.getElementById('noteStatus').value;
            const isPinned = document.getElementById('notePinned').checked;
            const classSlot = document.getElementById('noteClassSlot').value.trim();
            const instructor = document.getElementById('noteInstructor').value.trim();

            if (!title || !dueDate) {
                showToast('Vui lòng nhập tiêu đề và chọn ngày đến hạn!', 'warning');
                return;
            }

            const now = Date.now();
            notes.forEach(n => n.order = (n.order ?? 0) + 1);

            const newNote = {
                id: 'note_' + now + '_' + Math.random().toString(36).substring(2, 6),
                title,
                content,
                createdAt: now,
                updatedAt: now,
                dueDate,
                reminderTime,
                category,
                priority,
                status,
                isPinned,
                repeatDays,
                classSlot,
                instructor,
                notified: false,
                lastNotifiedDate: '',
                order: 0
            };

            notes.unshift(newNote);
            saveToStorage();
            resetCreateForm();
            renderAll();
            showToast('Đã tạo ghi chú / lịch học mới thành công!', 'success');
        }

        function resetCreateForm() {
            document.getElementById('addNoteForm').reset();
            setSelectedDaysInPicker('createDaysPicker', []);
            const today = getTodayISO();
            document.getElementById('noteDueDate').value = today;
            document.getElementById('formattedDatePreview').textContent = `Deadline: ${formatDateVN(today)}`;
        }

        // Chuyển đổi nhanh trạng thái (Chưa làm -> Đang làm -> Hoàn thành) hoặc chọn trực tiếp
        function changeNoteStatus(id, newStatus) {
            const note = notes.find(n => n.id === id);
            if (!note || !['todo', 'in_progress', 'done'].includes(newStatus)) return;
            note.status = newStatus;
            note.updatedAt = Date.now();
            saveToStorage();
            renderAll();

            const statusLabels = {
                todo: '⏳ Chưa làm',
                in_progress: '⚡ Đang làm',
                done: '✅ Hoàn thành'
            };
            showToast(`Đã chuyển trạng thái sang "${statusLabels[newStatus]}"`, 'success');
        }

        // Ghim / Bỏ ghim yêu thích
        function togglePinNote(id) {
            const note = notes.find(n => n.id === id);
            if (!note) return;
            note.isPinned = !note.isPinned;
            note.updatedAt = Date.now();
            saveToStorage();
            renderAll();
            showToast(note.isPinned ? '📌 Đã ghim ghi chú lên đầu!' : 'Đã bỏ ghim yêu thích.', 'success');
        }

        function deleteNote(id) {
            const index = notes.findIndex(n => n.id === id);
            if (index === -1) return;

            lastDeletedNote = { ...notes[index], originalIndex: index };
            notes.splice(index, 1);
            saveToStorage();
            renderAll();

            // Hiển thị Toast có nút Hoàn tác (Undo)
            showUndoToast(`Đã xóa ghi chú "${lastDeletedNote.title}"`);
        }

        function undoDeleteNote() {
            if (!lastDeletedNote) return;
            const { originalIndex, ...restoredNote } = lastDeletedNote;
            notes.splice(Math.min(originalIndex, notes.length), 0, restoredNote);
            lastDeletedNote = null;
            saveToStorage();
            renderAll();
            showToast('Đã khôi phục ghi chú vừa xóa!', 'success');
        }

        // Mở Modal Chỉnh sửa
        function openEditModal(id) {
            const note = notes.find(n => n.id === id);
            if (!note) return;

            document.getElementById('editNoteId').value = note.id;
            document.getElementById('editNoteTitle').value = note.title;
            document.getElementById('editNoteContent').value = note.content || '';
            document.getElementById('editNoteDueDate').value = note.dueDate;
            document.getElementById('editFormattedDatePreview').textContent = `Deadline: ${formatDateVN(note.dueDate)}`;
            document.getElementById('editNoteReminderTime').value = note.reminderTime || '';
            document.getElementById('editNoteCategory').value = note.category || 'Cá nhân';
            document.getElementById('editNotePriority').value = note.priority || 'normal';
            document.getElementById('editNoteStatus').value = note.status || 'todo';
            document.getElementById('editNotePinned').checked = Boolean(note.isPinned);
            setSelectedDaysInPicker('editDaysPicker', note.repeatDays || []);
            document.getElementById('editNoteClassSlot').value = note.classSlot || '';
            document.getElementById('editNoteInstructor').value = note.instructor || '';

            document.getElementById('editMetaTimestamps').textContent =
                `Tạo lúc: ${formatDateTimeVN(note.createdAt)} • Cập nhật gần nhất: ${formatDateTimeVN(note.updatedAt)}`;

            document.getElementById('editModal').classList.remove('hidden');
        }

        function closeEditModal() {
            document.getElementById('editModal').classList.add('hidden');
        }

        function handleUpdateNote(e) {
            e.preventDefault();
            const id = document.getElementById('editNoteId').value;
            const note = notes.find(n => n.id === id);
            if (!note) return;

            const newDueDate = document.getElementById('editNoteDueDate').value;
            const newReminderTime = document.getElementById('editNoteReminderTime').value;

            if (note.dueDate !== newDueDate || note.reminderTime !== newReminderTime) {
                note.notified = false;
                note.lastNotifiedDate = '';
            }

            note.title = document.getElementById('editNoteTitle').value.trim();
            note.content = document.getElementById('editNoteContent').value.trim();
            note.dueDate = newDueDate;
            note.reminderTime = newReminderTime;
            note.category = document.getElementById('editNoteCategory').value;
            note.priority = document.getElementById('editNotePriority').value;
            note.status = document.getElementById('editNoteStatus').value;
            note.isPinned = document.getElementById('editNotePinned').checked;
            note.repeatDays = getSelectedDaysFromPicker('editDaysPicker');
            note.classSlot = document.getElementById('editNoteClassSlot').value.trim();
            note.instructor = document.getElementById('editNoteInstructor').value.trim();
            note.updatedAt = Date.now();

            saveToStorage();
            closeEditModal();
            renderAll();
            showToast('Đã cập nhật ghi chú!', 'success');
        }

        /**
         * ====================================================================
         * 5. SẮP XẾP THỨ TỰ (THỦ CÔNG KÉO THẢ / NÚT LÊN XUỐNG & TỰ ĐỘNG)
         * ====================================================================
         */
        function handleSortChange() {
            currentSort = document.getElementById('sortSelect').value;
            localStorage.setItem(SORT_PREF_KEY, currentSort);

            const hint = document.getElementById('manualSortHint');
            if (currentSort === 'manual') {
                hint.classList.remove('opacity-40');
            } else {
                hint.classList.add('opacity-40');
            }
            renderNotes();
        }

        // Di chuyển vị trí thủ công bằng nút mũi tên Lên / Xuống
        function moveNoteOrder(id, direction) {
            // Chuyển sang chế độ sắp xếp thủ công nếu đang ở chế độ khác
            if (currentSort !== 'manual') {
                currentSort = 'manual';
                document.getElementById('sortSelect').value = 'manual';
                localStorage.setItem(SORT_PREF_KEY, 'manual');
            }

            // Sắp xếp mảng gốc theo order
            notes.sort((a, b) => (a.order ?? 0) - (b.order ?? 0));
            const idx = notes.findIndex(n => n.id === id);
            if (idx === -1) return;

            const targetIdx = direction === 'up' ? idx - 1 : idx + 1;
            if (targetIdx < 0 || targetIdx >= notes.length) return;

            // Hoán đổi vị trí
            const temp = notes[idx];
            notes[idx] = notes[targetIdx];
            notes[targetIdx] = temp;

            // Cập nhật lại chỉ số order chuẩn
            notes.forEach((n, i) => n.order = i);
            saveToStorage();
            renderNotes();
        }

        // Kéo thả thẻ (Drag & Drop)
        function handleDragStart(e, id) {
            draggedNoteId = id;
            e.currentTarget.classList.add('dragging');
            e.dataTransfer.effectAllowed = 'move';
        }

        function handleDragEnd(e) {
            e.currentTarget.classList.remove('dragging');
            document.querySelectorAll('.note-card').forEach(card => card.classList.remove('drag-over'));
        }

        function handleDragOver(e) {
            e.preventDefault();
            e.dataTransfer.dropEffect = 'move';
            e.currentTarget.classList.add('drag-over');
        }

        function handleDragLeave(e) {
            e.currentTarget.classList.remove('drag-over');
        }

        function handleDrop(e, targetId) {
            e.preventDefault();
            e.currentTarget.classList.remove('drag-over');
            if (!draggedNoteId || draggedNoteId === targetId) return;

            // Tự động chuyển sang chế độ thủ công để lưu thứ tự kéo thả
            if (currentSort !== 'manual') {
                currentSort = 'manual';
                document.getElementById('sortSelect').value = 'manual';
                localStorage.setItem(SORT_PREF_KEY, 'manual');
            }

            notes.sort((a, b) => (a.order ?? 0) - (b.order ?? 0));
            const fromIndex = notes.findIndex(n => n.id === draggedNoteId);
            const toIndex = notes.findIndex(n => n.id === targetId);

            if (fromIndex !== -1 && toIndex !== -1) {
                const [movedItem] = notes.splice(fromIndex, 1);
                notes.splice(toIndex, 0, movedItem);
                notes.forEach((n, i) => n.order = i);
                saveToStorage();
                renderNotes();
                showToast('Đã cập nhật thứ tự ghi chú.', 'success');
            }
            draggedNoteId = null;
        }

        /**
         * ====================================================================
         * 6. HIỂN THỊ GIAO DIỆN (RENDER CARDS, UPCOMING REMINDERS, STATS)
         * ====================================================================
         */
        function togglePinnedFilter() {
            onlyPinnedFilter = !onlyPinnedFilter;
            const btn = document.getElementById('btnFilterPinned');
            if (onlyPinnedFilter) {
                btn.className = 'px-3 py-2 rounded-xl bg-forest-900 text-nude-50 text-xs font-semibold transition flex items-center justify-center gap-1.5 cursor-pointer shadow-xs';
            } else {
                btn.className = 'px-3 py-2 rounded-xl bg-nude-200 hover:bg-nude-300 text-forest-900 text-xs font-semibold transition flex items-center justify-center gap-1.5 cursor-pointer';
            }
            renderNotes();
        }

        function resetAllFilters() {
            document.getElementById('searchInput').value = '';
            document.getElementById('filterStatus').value = 'all';
            document.getElementById('filterCategory').value = 'all';
            document.getElementById('filterPriority').value = 'all';
            onlyPinnedFilter = false;
            document.getElementById('btnFilterPinned').className =
                'px-3 py-2 rounded-xl bg-nude-200 hover:bg-nude-300 text-forest-900 text-xs font-semibold transition flex items-center justify-center gap-1.5 cursor-pointer';
            renderNotes();
        }

        function renderUpcomingReminders() {
            const container = document.getElementById('upcomingRemindersList');
            const badge = document.getElementById('upcomingCountBadge');
            const today = getTodayISO();
            const todayDow = new Date().getDay();

            const upcoming = notes
                .filter(n => {
                    if (n.status === 'done' || !n.reminderTime) return false;
                    const isRecurringToday = Array.isArray(n.repeatDays) && n.repeatDays.includes(todayDow);
                    return isRecurringToday || n.dueDate >= today;
                })
                .map(n => {
                    const isRecurringToday = Array.isArray(n.repeatDays) && n.repeatDays.includes(todayDow);
                    return { ...n, effectiveDate: isRecurringToday ? today : n.dueDate };
                })
                .sort((a, b) => `${a.effectiveDate} ${a.reminderTime}`.localeCompare(`${b.effectiveDate} ${b.reminderTime}`))
                .slice(0, 6);

            badge.textContent = `${upcoming.length} lịch hẹn`;

            if (upcoming.length === 0) {
                container.innerHTML = `
                    <div class="col-span-full text-xs text-nude-800 bg-nude-100/80 border border-nude-200 rounded-xl px-4 py-3 flex items-center gap-2">
                        <i class="ph ph-sparkle text-base text-forest-700"></i>
                        Không có lịch nhắc nhở nào sắp tới. Hãy chọn <strong>Ngày đến hạn</strong> và <strong>Giờ nhắc nhở</strong> để đặt hẹn giờ!
                    </div>
                `;
                return;
            }

            container.innerHTML = upcoming.map(note => {
                const isToday = note.effectiveDate === today;
                const repeatLabel = formatRepeatDaysVN(note.repeatDays);
                return `
                    <div class="bg-white border ${isToday ? 'border-nude-400 ring-1 ring-nude-400/40' : 'border-nude-200'} rounded-xl p-3 flex items-center justify-between gap-2.5 shadow-xs">
                        <div class="min-w-0">
                            <div class="flex flex-wrap items-center gap-1.5 mb-1">
                                <span class="text-[11px] font-bold px-2 py-0.5 rounded-md ${isToday ? 'bg-forest-900 text-nude-100' : 'bg-nude-200 text-forest-900'}">
                                    ${isToday ? 'Hôm nay' : formatDateVN(note.effectiveDate)} • ${note.reminderTime}
                                </span>
                                ${repeatLabel ? `<span class="text-[10px] font-semibold px-1.5 py-0.5 rounded bg-nude-200 text-forest-900">${escapeHTML(repeatLabel)}</span>` : ''}
                            </div>
                            <p class="text-sm font-semibold text-forest-950 truncate" title="${escapeHTML(note.title)}">
                                ${escapeHTML(note.title)}
                            </p>
                        </div>
                        <button onclick="changeNoteStatus('${note.id}', 'done')" title="Đánh dấu hoàn thành"
                            class="w-8 h-8 rounded-lg bg-forest-50 hover:bg-forest-900 text-forest-800 hover:text-nude-50 flex items-center justify-center shrink-0 transition cursor-pointer">
                            <i class="ph-bold ph-check text-sm"></i>
                        </button>
                    </div>
                `;
            }).join('');
        }

        function renderStats() {
            const total = notes.length;
            const inProgress = notes.filter(n => n.status === 'in_progress').length;
            const pinned = notes.filter(n => n.isPinned).length;
            const completed = notes.filter(n => n.status === 'done').length;

            document.getElementById('statTotal').textContent = total;
            document.getElementById('statInProgress').textContent = inProgress;
            document.getElementById('statPinned').textContent = pinned;
            document.getElementById('statCompleted').textContent = completed;
        }

        function renderNotes() {
            const container = document.getElementById('notesContainer');
            const emptyState = document.getElementById('emptyState');
            const keyword = document.getElementById('searchInput').value.trim().toLowerCase();
            const statusFilter = document.getElementById('filterStatus').value;
            const categoryFilter = document.getElementById('filterCategory').value;
            const priorityFilter = document.getElementById('filterPriority').value;

            // 1. Lọc dữ liệu kết hợp
            let filtered = notes.filter(note => {
                if (statusFilter !== 'all' && note.status !== statusFilter) return false;
                if (categoryFilter !== 'all' && note.category !== categoryFilter) return false;
                if (priorityFilter !== 'all' && note.priority !== priorityFilter) return false;
                if (onlyPinnedFilter && !note.isPinned) return false;

                if (keyword) {
                    const matchTitle = note.title.toLowerCase().includes(keyword);
                    const matchContent = (note.content || '').toLowerCase().includes(keyword);
                    const matchCat = (note.category || '').toLowerCase().includes(keyword);
                    const matchDue = formatDateVN(note.dueDate).includes(keyword);
                    return matchTitle || matchContent || matchCat || matchDue;
                }
                return true;
            });

            // 2. Sắp xếp dữ liệu: Luôn ưu tiên Ghim yêu thích lên đầu, sau đó mới áp dụng tiêu chí sắp xếp
            const priorityWeight = { high: 3, normal: 2, low: 1 };
            filtered.sort((a, b) => {
                if (a.isPinned !== b.isPinned) {
                    return a.isPinned ? -1 : 1;
                }

                if (currentSort === 'manual') {
                    return (a.order ?? 0) - (b.order ?? 0);
                } else if (currentSort === 'due_asc') {
                    return `${a.dueDate} ${a.reminderTime || '23:59'}`.localeCompare(`${b.dueDate} ${b.reminderTime || '23:59'}`);
                } else if (currentSort === 'due_desc') {
                    return `${b.dueDate} ${b.reminderTime || '00:00'}`.localeCompare(`${a.dueDate} ${a.reminderTime || '00:00'}`);
                } else if (currentSort === 'created_desc') {
                    return (b.createdAt || 0) - (a.createdAt || 0);
                } else if (currentSort === 'updated_desc') {
                    return (b.updatedAt || 0) - (a.updatedAt || 0);
                } else if (currentSort === 'priority_desc') {
                    return (priorityWeight[b.priority] || 0) - (priorityWeight[a.priority] || 0);
                } else if (currentSort === 'priority_asc') {
                    return (priorityWeight[a.priority] || 0) - (priorityWeight[b.priority] || 0);
                }
                return 0;
            });

            // 3. Hiển thị Empty State nếu không có ghi chú
            if (filtered.length === 0) {
                container.innerHTML = '';
                emptyState.classList.remove('hidden');
                return;
            }
            emptyState.classList.add('hidden');

            const today = getTodayISO();

            // 4. Render từng thẻ ghi chú với đầy đủ 10 trường thông tin
            container.innerHTML = filtered.map((note, idx) => {
                const isDone = note.status === 'done';
                const isOverdue = !isDone && note.dueDate < today;
                const isDueToday = !isDone && note.dueDate === today;

                // Badge Mức độ ưu tiên (Thấp, Bình thường, Cao)
                let priorityBadge = '';
                if (note.priority === 'high') {
                    priorityBadge = `<span class="px-2.5 py-0.5 rounded-full text-[11px] font-semibold bg-red-100 text-red-800 border border-red-200">🔴 Cao</span>`;
                } else if (note.priority === 'normal') {
                    priorityBadge = `<span class="px-2.5 py-0.5 rounded-full text-[11px] font-semibold bg-nude-200 text-nude-900 border border-nude-300">🟡 Bình thường</span>`;
                } else {
                    priorityBadge = `<span class="px-2.5 py-0.5 rounded-full text-[11px] font-semibold bg-forest-100 text-forest-800 border border-forest-200">🟢 Thấp</span>`;
                }

                // Màu sắc dropdown Trạng thái
                let statusSelectStyle = 'bg-nude-200 text-forest-950 border-nude-300';
                if (note.status === 'in_progress') {
                    statusSelectStyle = 'bg-amber-100 text-amber-900 border-amber-300';
                } else if (note.status === 'done') {
                    statusSelectStyle = 'bg-forest-100 text-forest-900 border-forest-300';
                }

                return `
                    <article
                        draggable="true"
                        ondragstart="handleDragStart(event, '${note.id}')"
                        ondragend="handleDragEnd(event)"
                        ondragover="handleDragOver(event)"
                        ondragleave="handleDragLeave(event)"
                        ondrop="handleDrop(event, '${note.id}')"
                        class="note-card group bg-nude-50 hover:bg-white border ${note.isPinned ? 'border-nude-400 ring-1 ring-nude-400/50' : (isDone ? 'border-nude-200 opacity-80' : 'border-nude-300 hover:border-forest-600')} rounded-2xl p-4 sm:p-5 shadow-card transition-all flex flex-col justify-between gap-3.5 relative">

                        <div>
                            <!-- Hàng 1: Danh mục, Mức độ ưu tiên, Ghim & Nút đổi thứ tự -->
                            <div class="flex items-center justify-between gap-2 mb-2.5">
                                <div class="flex flex-wrap items-center gap-1.5">
                                    <span class="px-2.5 py-0.5 rounded-full text-[11px] font-semibold bg-forest-900 text-nude-100">
                                        ${escapeHTML(note.category || 'Cá nhân')}
                                    </span>
                                    ${priorityBadge}
                                </div>

                                <div class="flex items-center gap-1 shrink-0">
                                    <!-- Nút Ghim yêu thích -->
                                    <button onclick="togglePinNote('${note.id}')"
                                        title="${note.isPinned ? 'Bỏ ghim yêu thích' : 'Ghim yêu thích lên đầu'}"
                                        class="p-1.5 rounded-lg ${note.isPinned ? 'bg-nude-400 text-forest-950 shadow-xs' : 'bg-nude-200/70 hover:bg-nude-300 text-nude-800'} transition cursor-pointer">
                                        <i class="${note.isPinned ? 'ph-fill' : 'ph'} ph-push-pin text-sm"></i>
                                    </button>

                                    <!-- Nút điều chỉnh thứ tự lên / xuống -->
                                    <div class="flex items-center gap-0.5 bg-nude-200/70 rounded-lg p-0.5" title="Đổi thứ tự hiển thị">
                                        <button onclick="moveNoteOrder('${note.id}', 'up')" ${idx === 0 ? 'disabled' : ''}
                                            class="p-1 rounded-md hover:bg-white text-forest-900 disabled:opacity-30 transition cursor-pointer">
                                            <i class="ph-bold ph-caret-up text-xs"></i>
                                        </button>
                                        <button onclick="moveNoteOrder('${note.id}', 'down')" ${idx === filtered.length - 1 ? 'disabled' : ''}
                                            class="p-1 rounded-md hover:bg-white text-forest-900 disabled:opacity-30 transition cursor-pointer">
                                            <i class="ph-bold ph-caret-down text-xs"></i>
                                        </button>
                                        <span class="p-1 text-nude-800 cursor-grab active:cursor-grabbing" title="Kéo thả thẻ">
                                            <i class="ph-bold ph-dots-six-vertical text-xs"></i>
                                        </span>
                                    </div>
                                </div>
                            </div>

                            <!-- Hàng 2: Tiêu đề ghi chú -->
                            <h3 class="font-display font-bold text-base sm:text-lg text-forest-950 leading-snug mb-1.5 ${isDone ? 'line-through text-nude-800' : ''}">
                                ${escapeHTML(note.title)}
                            </h3>

                            <!-- Hàng 3: Nội dung chi tiết -->
                            <p class="text-sm text-forest-900/85 whitespace-pre-line leading-relaxed break-words">
                                ${note.content ? escapeHTML(note.content) : '<span class="italic text-nude-800/60">Không có mô tả chi tiết</span>'}
                            </p>

                            ${(note.repeatDays?.length || note.classSlot || note.instructor) ? `
                                <div class="mt-2.5 flex flex-wrap items-center gap-1.5 text-[11px] font-semibold">
                                    ${note.repeatDays?.length ? `
                                        <span class="inline-flex items-center gap-1 px-2.5 py-1 rounded-lg bg-forest-100 text-forest-900 border border-forest-600/20">
                                            <i class="ph-fill ph-calendar-blank text-forest-700"></i>
                                            ${escapeHTML(formatRepeatDaysVN(note.repeatDays))}
                                        </span>
                                    ` : ''}
                                    ${note.classSlot ? `
                                        <span class="inline-flex items-center gap-1 px-2.5 py-1 rounded-lg bg-nude-200 text-forest-950 border border-nude-300">
                                            <i class="ph-fill ph-clock text-forest-700"></i>
                                            ${escapeHTML(note.classSlot)}
                                        </span>
                                    ` : ''}
                                    ${note.instructor ? `
                                        <span class="inline-flex items-center gap-1 px-2.5 py-1 rounded-lg bg-nude-400/30 text-forest-950 border border-nude-400/50">
                                            <i class="ph-fill ph-chalkboard-teacher text-nude-500"></i>
                                            ${escapeHTML(note.instructor)}
                                        </span>
                                    ` : ''}
                                </div>
                            ` : ''}
                        </div>

                        <!-- Phần thông tin thời gian & hành động -->
                        <div class="space-y-2.5 pt-3 border-t border-nude-200">
                            <!-- Hàng 4: Ngày đến hạn (Deadline) & Giờ nhắc nhở -->
                            <div class="flex flex-wrap items-center gap-1.5 text-xs font-medium">
                                <span class="inline-flex items-center gap-1 px-2.5 py-1 rounded-lg ${isOverdue ? 'bg-red-100 text-red-800 font-semibold' : (isDueToday ? 'bg-amber-100 text-amber-900 font-semibold' : 'bg-nude-200/80 text-forest-900')}">
                                    <i class="ph-fill ph-calendar-check text-forest-700"></i>
                                    Hạn: ${formatDateVN(note.dueDate)}
                                    ${isOverdue ? '(Quá hạn)' : (isDueToday ? '(Hôm nay)' : '')}
                                </span>

                                ${note.reminderTime ? `
                                    <span class="inline-flex items-center gap-1 px-2.5 py-1 rounded-lg ${isDone ? 'bg-nude-200 text-nude-800' : 'bg-nude-400/25 text-forest-950 border border-nude-400/50'}">
                                        <i class="ph-fill ph-alarm text-nude-500"></i>
                                        Nhắc: ${note.reminderTime}
                                    </span>
                                ` : ''}
                            </div>

                            <!-- Hàng 5: Ngày tạo, Ngày cập nhật & Chủ lưu trú -->
                            <div class="flex flex-wrap items-center justify-between gap-x-3 gap-y-1 text-[11px] text-nude-800 bg-nude-100/70 px-2.5 py-1.5 rounded-lg">
                                <span title="Thời điểm tạo ghi chú">
                                    <i class="ph ph-clock-clockwise"></i> Tạo: ${formatDateTimeVN(note.createdAt)}
                                </span>
                                <span title="Lần chỉnh sửa gần nhất">
                                    <i class="ph ph-pencil-line"></i> Sửa: ${formatDateTimeVN(note.updatedAt)}
                                </span>
                                <span title="Chủ lưu trú" class="font-medium text-forest-800">
                                    <i class="ph-fill ph-user"></i> ${escapeHTML(ownerName)}
                                </span>
                            </div>

                            <!-- Hàng 6: Bộ chọn Trạng thái (Chưa làm / Đang làm / Hoàn thành) & Nút Sửa / Xóa -->
                            <div class="flex items-center justify-between gap-2 pt-0.5">
                                <select onchange="changeNoteStatus('${note.id}', this.value)" aria-label="Đổi trạng thái ghi chú"
                                    class="px-2.5 py-1.5 rounded-xl border text-xs font-semibold focus:outline-none focus:ring-2 focus:ring-forest-700 transition cursor-pointer ${statusSelectStyle}">
                                    <option value="todo" ${note.status === 'todo' ? 'selected' : ''}>⏳ Chưa làm</option>
                                    <option value="in_progress" ${note.status === 'in_progress' ? 'selected' : ''}>⚡ Đang làm</option>
                                    <option value="done" ${note.status === 'done' ? 'selected' : ''}>✅ Hoàn thành</option>
                                </select>

                                <div class="flex items-center gap-1.5">
                                    <button onclick="openEditModal('${note.id}')" title="Chỉnh sửa ghi chú"
                                        class="p-2 rounded-xl bg-nude-200 hover:bg-nude-400 text-forest-950 transition cursor-pointer">
                                        <i class="ph-bold ph-pencil-simple text-sm"></i>
                                    </button>

                                    <button onclick="deleteNote('${note.id}')" title="Xóa ghi chú"
                                        class="p-2 rounded-xl bg-red-50 hover:bg-red-600 text-red-700 hover:text-white transition cursor-pointer">
                                        <i class="ph-bold ph-trash text-sm"></i>
                                    </button>
                                </div>
                            </div>
                        </div>
                    </article>
                `;
            }).join('');
        }

        function renderAll() {
            renderStats();
            renderUpcomingReminders();
            renderNotes();
        }

        /**
         * ====================================================================
         * 7. SAO LƯU & KHÔI PHỤC DỮ LIỆU (EXPORT / IMPORT JSON)
         * ====================================================================
         */
        function exportDataJSON() {
            try {
                const payload = {
                    app: 'EmeraldNote Personal Manager',
                    ownerName: ownerName,
                    exportedAt: new Date().toISOString(),
                    totalNotes: notes.length,
                    notes: notes
                };
                const blob = new Blob([JSON.stringify(payload, null, 2)], { type: 'application/json' });
                const url = URL.createObjectURL(blob);
                const a = document.createElement('a');
                a.href = url;
                a.download = `emeraldnote_backup_${getTodayISO()}.json`;
                document.body.appendChild(a);
                a.click();
                document.body.removeChild(a);
                URL.revokeObjectURL(url);
                showToast('Đã tải xuống file sao lưu JSON!', 'success');
            } catch (err) {
                console.error(err);
                showToast('Có lỗi khi xuất dữ liệu JSON.', 'error');
            }
        }

        function importDataJSON(event) {
            const file = event.target.files[0];
            if (!file) return;

            const reader = new FileReader();
            reader.onload = function (e) {
                try {
                    const parsed = JSON.parse(e.target.result);
                    const importedNotes = Array.isArray(parsed) ? parsed : parsed.notes;

                    if (!Array.isArray(importedNotes)) {
                        throw new Error('Cấu trúc file JSON không hợp lệ.');
                    }

                    if (parsed && typeof parsed.ownerName === 'string' && parsed.ownerName.trim()) {
                        ownerName = parsed.ownerName.trim();
                        localStorage.setItem(OWNER_NAME_KEY, ownerName);
                        loadOwnerNameUI();
                    }

                    // Hợp nhất hoặc thay thế dữ liệu hợp lệ (chuẩn hóa đủ 10 trường)
                    const validNotes = importedNotes
                        .filter(n => n && n.title && (n.dueDate || n.date))
                        .map((n, idx) => normalizeNoteSchema(n, idx));

                    notes = validNotes;
                    saveToStorage();
                    renderAll();
                    showToast(`Đã khôi phục thành công ${validNotes.length} ghi chú!`, 'success');
                } catch (err) {
                    console.error(err);
                    showToast('File JSON không hợp lệ hoặc bị hỏng.', 'error');
                } finally {
                    event.target.value = '';
                }
            };
            reader.readAsText(file);
        }

        /**
         * ====================================================================
         * 8. TIỆN ÍCH GIAO DIỆN (TOAST, ESCAPE HTML, PREVIEW NGÀY)
         * ====================================================================
         */
        function escapeHTML(str) {
            return String(str)
                .replace(/&/g, '&amp;')
                .replace(/</g, '&lt;')
                .replace(/>/g, '&gt;')
                .replace(/"/g, '&quot;')
                .replace(/'/g, '&#039;');
        }

        function showToast(message, type = 'success') {
            const container = document.getElementById('toastContainer');
            const toast = document.createElement('div');

            let bgClass = 'bg-forest-900 text-nude-50 border-forest-700';
            let iconClass = 'ph-fill ph-check-circle text-nude-400';

            if (type === 'warning') {
                bgClass = 'bg-nude-400 text-forest-950 border-nude-500';
                iconClass = 'ph-fill ph-warning-circle text-forest-950';
            } else if (type === 'error') {
                bgClass = 'bg-red-800 text-white border-red-600';
                iconClass = 'ph-fill ph-x-circle text-red-200';
            } else if (type === 'reminder') {
                bgClass = 'bg-forest-950 text-nude-100 border-nude-400 ring-2 ring-nude-400';
                iconClass = 'ph-fill ph-bell-ringing text-nude-400 animate-ring';
            }

            toast.className = `pointer-events-auto flex items-center gap-2.5 px-4 py-3 rounded-xl border shadow-xl text-xs sm:text-sm font-medium transition-all transform translate-y-2 opacity-0 ${bgClass}`;
            toast.innerHTML = `
                <i class="${iconClass} text-lg shrink-0"></i>
                <span class="flex-1">${escapeHTML(message)}</span>
            `;

            container.appendChild(toast);
            requestAnimationFrame(() => {
                toast.classList.remove('translate-y-2', 'opacity-0');
            });

            setTimeout(() => {
                toast.classList.add('opacity-0', 'translate-y-2');
                setTimeout(() => toast.remove(), 300);
            }, 3800);
        }

        function showUndoToast(message) {
            const container = document.getElementById('toastContainer');
            const toast = document.createElement('div');
            toast.className = 'pointer-events-auto flex items-center justify-between gap-3 px-4 py-3 rounded-xl border border-forest-700 bg-forest-900 text-nude-50 shadow-xl text-xs sm:text-sm font-medium transition-all';
            toast.innerHTML = `
                <div class="flex items-center gap-2 truncate">
                    <i class="ph-fill ph-trash text-nude-400 text-lg shrink-0"></i>
                    <span class="truncate">${escapeHTML(message)}</span>
                </div>
                <button onclick="undoDeleteNote(); this.parentElement.remove();"
                    class="px-2.5 py-1 rounded-lg bg-nude-400 hover:bg-nude-500 text-forest-950 font-bold text-xs shrink-0 transition cursor-pointer">
                    Hoàn tác
                </button>
            `;
            container.appendChild(toast);
            setTimeout(() => {
                if (toast.parentElement) {
                    toast.classList.add('opacity-0');
                    setTimeout(() => toast.remove(), 300);
                }
            }, 5000);
        }

        // Lắng nghe thay đổi input date để hiển thị bản xem trước dd/mm/yyyy rõ ràng
        function setupDatePreviews() {
            const createDateInput = document.getElementById('noteDueDate');
            const createPreview = document.getElementById('formattedDatePreview');
            createDateInput.addEventListener('input', () => {
                createPreview.textContent = `Deadline: ${formatDateVN(createDateInput.value)}`;
            });

            const editDateInput = document.getElementById('editNoteDueDate');
            const editPreview = document.getElementById('editFormattedDatePreview');
            editDateInput.addEventListener('input', () => {
                editPreview.textContent = `Deadline: ${formatDateVN(editDateInput.value)}`;
            });
        }

        /**
         * ====================================================================
         * 9. KHỞI CHẠY ỨNG DỤNG KHI TRANG TẢI XONG
         * ====================================================================
         */
        window.addEventListener('DOMContentLoaded', () => {
            loadOwnerNameUI();
            loadFromStorage();

            // Khôi phục lựa chọn sắp xếp đã lưu
            document.getElementById('sortSelect').value = currentSort;
            if (currentSort !== 'manual') {
                document.getElementById('manualSortHint').classList.add('opacity-40');
            }

            // Đặt ngày mặc định là hôm nay cho form tạo mới
            resetCreateForm();
            setupDatePreviews();
            updateNotifButtonUI();

            // Render giao diện lần đầu
            renderAll();

            // Khởi động đồng hồ thời gian thực & kiểm tra nhắc nhở mỗi 1 giây
            updateLiveClockAndReminders();
            setInterval(updateLiveClockAndReminders, 1000);
        });
    </script>
</body>
</html>
