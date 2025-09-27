<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>College Attendance Calendar - Track Your Daily Progress</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
    <style>
        @import url('https://fonts.googleapis.com/css2?family=Inter:wght@300;400;500;600;700&display=swap');
        
        * {
            font-family: 'Inter', sans-serif;
        }
        
        .gradient-bg {
            background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
        }
        
        .card-shadow {
            box-shadow: 0 10px 25px rgba(0,0,0,0.1);
            transition: all 0.3s ease;
        }
        
        .card-shadow:hover {
            transform: translateY(-2px);
            box-shadow: 0 15px 35px rgba(0,0,0,0.15);
        }
        
        .calendar-day {
            min-height: 120px;
            transition: all 0.2s ease;
        }
        
        .calendar-day:hover {
            background-color: #f8fafc;
        }
        
        /* Mobile responsive calendar */
        @media (max-width: 768px) {
            .calendar-day {
                min-height: 80px;
                padding: 8px 4px;
            }
            
            .day-number {
                font-size: 14px;
                line-height: 1;
                white-space: nowrap;
            }
            
            .day-summary {
                font-size: 9px;
                padding: 1px 3px;
                margin-top: 2px;
                line-height: 1.2;
            }
        }
        
        @media (max-width: 480px) {
            .calendar-day {
                min-height: 70px;
                padding: 6px 2px;
            }
            
            .day-number {
                font-size: 12px;
            }
            
            .day-summary {
                font-size: 8px;
                padding: 1px 2px;
            }
        }
        
        .working-day {
            background: linear-gradient(135deg, #f0f9ff, #e0f2fe);
            border: 2px solid #0ea5e9;
        }
        
        .holiday {
            background: linear-gradient(135deg, #fef3c7, #fde68a);
            border: 2px solid #f59e0b;
        }
        
        .lecture-checkbox {
            width: 16px;
            height: 16px;
            accent-color: #10b981;
        }
        
        .progress-circle {
            transform: rotate(-90deg);
        }
        
        .btn-primary {
            background: linear-gradient(135deg, #3b82f6, #1d4ed8);
            transition: all 0.3s ease;
        }
        
        .btn-primary:hover {
            transform: translateY(-2px);
            box-shadow: 0 8px 20px rgba(59, 130, 246, 0.3);
        }
        
        .btn-secondary {
            background: linear-gradient(135deg, #10b981, #059669);
            transition: all 0.3s ease;
        }
        
        .btn-secondary:hover {
            transform: translateY(-2px);
            box-shadow: 0 8px 20px rgba(16, 185, 129, 0.3);
        }
        
        .stats-card {
            background: linear-gradient(135deg, #f0f9ff, #e0f2fe);
            border: 1px solid #bae6fd;
        }
        
        @keyframes fadeInUp {
            from {
                opacity: 0;
                transform: translateY(30px);
            }
            to {
                opacity: 1;
                transform: translateY(0);
            }
        }
        
        .fade-in-up {
            animation: fadeInUp 0.6s ease-out;
        }
        
        .modal-overlay {
            background: rgba(0, 0, 0, 0.5);
            backdrop-filter: blur(4px);
        }
        
        .attendance-excellent { color: #059669; }
        .attendance-good { color: #d97706; }
        .attendance-poor { color: #dc2626; }
        
        .day-summary {
            font-size: 10px;
            padding: 2px 4px;
            border-radius: 4px;
            margin-top: 4px;
        }
        
        .summary-excellent { background: #dcfce7; color: #166534; }
        .summary-good { background: #fef3c7; color: #92400e; }
        .summary-poor { background: #fee2e2; color: #991b1b; }
        .summary-none { background: #f1f5f9; color: #64748b; }
    </style>
</head>
<body class="bg-gray-50 min-h-screen">
    <!-- Header -->
    <header class="gradient-bg text-white py-6">
        <div class="max-w-7xl mx-auto px-4">
            <div class="flex flex-col md:flex-row md:items-center md:justify-between">
                <div>
                    <h1 class="text-2xl md:text-3xl font-bold mb-2">📅 College Attendance Calendar</h1>
                    <p class="text-blue-100">Track your daily attendance with ease</p>
                </div>
                
                <div class="mt-4 md:mt-0 flex space-x-3">
                    <button id="dashboardBtn" class="btn-secondary text-white px-4 py-2 rounded-lg font-medium">
                        📊 Dashboard
                    </button>
                    <button id="exportBtn" class="btn-primary text-white px-4 py-2 rounded-lg font-medium">
                        📤 Export Data
                    </button>
                </div>
            </div>
        </div>
    </header>

    <!-- Main Content -->
    <main class="max-w-7xl mx-auto px-4 py-8">
        <!-- Calendar Navigation -->
        <div class="bg-white rounded-xl card-shadow p-6 mb-8">
            <div class="flex flex-col sm:flex-row sm:items-center sm:justify-between mb-6">
                <h2 class="text-xl font-semibold text-gray-800 mb-4 sm:mb-0">Monthly Calendar</h2>
                
                <div class="flex items-center space-x-4">
                    <button id="prevMonth" class="p-2 hover:bg-gray-100 rounded-lg transition-colors">
                        <svg class="w-5 h-5 text-gray-600" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M15 19l-7-7 7-7"></path>
                        </svg>
                    </button>
                    
                    <h3 id="currentMonth" class="text-lg font-medium text-gray-800 min-w-[200px] text-center"></h3>
                    
                    <button id="nextMonth" class="p-2 hover:bg-gray-100 rounded-lg transition-colors">
                        <svg class="w-5 h-5 text-gray-600" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M9 5l7 7-7 7"></path>
                        </svg>
                    </button>
                </div>
            </div>
            
            <!-- Calendar Grid -->
            <div class="grid grid-cols-7 gap-1 md:gap-2">
                <!-- Day Headers -->
                <div class="text-center font-medium text-gray-600 py-2 md:py-3 text-sm md:text-base">Sun</div>
                <div class="text-center font-medium text-gray-600 py-2 md:py-3 text-sm md:text-base">Mon</div>
                <div class="text-center font-medium text-gray-600 py-2 md:py-3 text-sm md:text-base">Tue</div>
                <div class="text-center font-medium text-gray-600 py-2 md:py-3 text-sm md:text-base">Wed</div>
                <div class="text-center font-medium text-gray-600 py-2 md:py-3 text-sm md:text-base">Thu</div>
                <div class="text-center font-medium text-gray-600 py-2 md:py-3 text-sm md:text-base">Fri</div>
                <div class="text-center font-medium text-gray-600 py-2 md:py-3 text-sm md:text-base">Sat</div>
                
                <!-- Calendar Days -->
                <div id="calendarGrid" class="col-span-7 grid grid-cols-7 gap-1 md:gap-2">
                    <!-- Days will be populated here -->
                </div>
            </div>
        </div>

        <!-- Quick Stats -->
        <div class="grid grid-cols-2 md:grid-cols-5 gap-3 md:gap-4 mb-8">
            <div class="stats-card rounded-xl p-4 md:p-6 text-center">
                <div class="text-xl md:text-2xl mb-1 md:mb-2">📅</div>
                <div class="text-xl md:text-2xl font-bold text-blue-600" id="daysAttended">0</div>
                <div class="text-xs md:text-sm text-gray-600">Days Attended</div>
            </div>
            
            <div class="stats-card rounded-xl p-4 md:p-6 text-center">
                <div class="text-xl md:text-2xl mb-1 md:mb-2">📚</div>
                <div class="text-xl md:text-2xl font-bold text-gray-800" id="totalLectures">0</div>
                <div class="text-xs md:text-sm text-gray-600">Total Lectures</div>
            </div>
            
            <div class="stats-card rounded-xl p-4 md:p-6 text-center">
                <div class="text-xl md:text-2xl mb-1 md:mb-2">✅</div>
                <div class="text-xl md:text-2xl font-bold text-green-600" id="attendedLectures">0</div>
                <div class="text-xs md:text-sm text-gray-600">Lectures Attended</div>
            </div>
            
            <div class="stats-card rounded-xl p-4 md:p-6 text-center">
                <div class="text-xl md:text-2xl mb-1 md:mb-2">❌</div>
                <div class="text-xl md:text-2xl font-bold text-red-600" id="missedLectures">0</div>
                <div class="text-xs md:text-sm text-gray-600">Lectures Missed</div>
            </div>
            
            <div class="stats-card rounded-xl p-4 md:p-6 text-center col-span-2 md:col-span-1">
                <div class="text-xl md:text-2xl mb-1 md:mb-2">📈</div>
                <div class="text-xl md:text-2xl font-bold" id="attendancePercent">0%</div>
                <div class="text-xs md:text-sm text-gray-600">Attendance Rate</div>
            </div>
        </div>
    </main>

    <!-- Day Detail Modal -->
    <div id="dayModal" class="fixed inset-0 modal-overlay hidden items-center justify-center z-50 p-4">
        <div class="bg-white rounded-2xl card-shadow max-w-md w-full max-h-[90vh] overflow-y-auto">
            <div class="p-6">
                <div class="flex justify-between items-center mb-6">
                    <h3 id="modalTitle" class="text-xl font-semibold text-gray-800"></h3>
                    <button id="closeModal" class="p-2 hover:bg-gray-100 rounded-lg transition-colors">
                        <svg class="w-5 h-5 text-gray-600" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M6 18L18 6M6 6l12 12"></path>
                        </svg>
                    </button>
                </div>
                
                <!-- Day Type Selection -->
                <div class="mb-6">
                    <label class="block text-sm font-medium text-gray-700 mb-3">Day Type</label>
                    <div class="flex space-x-3">
                        <button id="workingDayBtn" class="flex-1 py-2 px-4 border-2 border-blue-200 text-blue-700 rounded-lg font-medium transition-all">
                            📚 Working Day
                        </button>
                        <button id="holidayBtn" class="flex-1 py-2 px-4 border-2 border-yellow-200 text-yellow-700 rounded-lg font-medium transition-all">
                            🏖️ Holiday
                        </button>
                    </div>
                </div>
                
                <!-- Days Attended Section -->
                <div id="attendanceSection" class="mb-6 hidden">
                    <label class="block text-sm font-medium text-gray-700 mb-3">College Attendance</label>
                    <div class="flex space-x-3">
                        <button id="presentBtn" class="flex-1 py-2 px-4 border-2 border-green-200 text-green-700 rounded-lg font-medium transition-all">
                            ✅ Present (Attended College)
                        </button>
                        <button id="absentBtn" class="flex-1 py-2 px-4 border-2 border-red-200 text-red-700 rounded-lg font-medium transition-all">
                            ❌ Absent (Didn't Attend College)
                        </button>
                    </div>
                </div>
                
                <!-- Lectures Section -->
                <div id="lecturesSection" class="hidden">
                    <label class="block text-sm font-medium text-gray-700 mb-3">Lectures Attendance</label>
                    <div class="space-y-3">
                        <div class="flex items-center justify-between p-3 bg-gray-50 rounded-lg">
                            <span class="font-medium text-gray-700">Lecture 1</span>
                            <div class="flex space-x-4">
                                <label class="flex items-center space-x-2">
                                    <input type="radio" name="lecture1" value="present" class="lecture-checkbox">
                                    <span class="text-green-600">Present</span>
                                </label>
                                <label class="flex items-center space-x-2">
                                    <input type="radio" name="lecture1" value="absent" class="lecture-checkbox">
                                    <span class="text-red-600">Absent</span>
                                </label>
                            </div>
                        </div>
                        
                        <div class="flex items-center justify-between p-3 bg-gray-50 rounded-lg">
                            <span class="font-medium text-gray-700">Lecture 2</span>
                            <div class="flex space-x-4">
                                <label class="flex items-center space-x-2">
                                    <input type="radio" name="lecture2" value="present" class="lecture-checkbox">
                                    <span class="text-green-600">Present</span>
                                </label>
                                <label class="flex items-center space-x-2">
                                    <input type="radio" name="lecture2" value="absent" class="lecture-checkbox">
                                    <span class="text-red-600">Absent</span>
                                </label>
                            </div>
                        </div>
                        
                        <div class="flex items-center justify-between p-3 bg-gray-50 rounded-lg">
                            <span class="font-medium text-gray-700">Lecture 3</span>
                            <div class="flex space-x-4">
                                <label class="flex items-center space-x-2">
                                    <input type="radio" name="lecture3" value="present" class="lecture-checkbox">
                                    <span class="text-green-600">Present</span>
                                </label>
                                <label class="flex items-center space-x-2">
                                    <input type="radio" name="lecture3" value="absent" class="lecture-checkbox">
                                    <span class="text-red-600">Absent</span>
                                </label>
                            </div>
                        </div>
                        
                        <div class="flex items-center justify-between p-3 bg-gray-50 rounded-lg">
                            <span class="font-medium text-gray-700">Lecture 4</span>
                            <div class="flex space-x-4">
                                <label class="flex items-center space-x-2">
                                    <input type="radio" name="lecture4" value="present" class="lecture-checkbox">
                                    <span class="text-green-600">Present</span>
                                </label>
                                <label class="flex items-center space-x-2">
                                    <input type="radio" name="lecture4" value="absent" class="lecture-checkbox">
                                    <span class="text-red-600">Absent</span>
                                </label>
                            </div>
                        </div>
                        
                        <div class="flex items-center justify-between p-3 bg-gray-50 rounded-lg">
                            <span class="font-medium text-gray-700">Lecture 5</span>
                            <div class="flex space-x-4">
                                <label class="flex items-center space-x-2">
                                    <input type="radio" name="lecture5" value="present" class="lecture-checkbox">
                                    <span class="text-green-600">Present</span>
                                </label>
                                <label class="flex items-center space-x-2">
                                    <input type="radio" name="lecture5" value="absent" class="lecture-checkbox">
                                    <span class="text-red-600">Absent</span>
                                </label>
                            </div>
                        </div>
                    </div>
                    
                    <!-- Quick Actions -->
                    <div class="flex space-x-3 mt-4">
                        <button id="markAllPresent" class="flex-1 btn-secondary text-white py-2 px-4 rounded-lg font-medium">
                            ✅ All Present
                        </button>
                        <button id="markAllAbsent" class="flex-1 bg-red-500 hover:bg-red-600 text-white py-2 px-4 rounded-lg font-medium transition-colors">
                            ❌ All Absent
                        </button>
                    </div>
                </div>
                
                <!-- Save Button -->
                <div class="mt-6">
                    <button id="saveDay" class="w-full btn-primary text-white py-3 px-4 rounded-lg font-medium">
                        💾 Save Changes
                    </button>
                </div>
            </div>
        </div>
    </div>

    <!-- Dashboard Modal -->
    <div id="dashboardModal" class="fixed inset-0 modal-overlay hidden items-center justify-center z-50 p-4">
        <div class="bg-white rounded-2xl card-shadow max-w-4xl w-full max-h-[90vh] overflow-y-auto">
            <div class="p-6">
                <div class="flex justify-between items-center mb-6">
                    <h3 class="text-2xl font-semibold text-gray-800">📊 Attendance Dashboard</h3>
                    <button id="closeDashboard" class="p-2 hover:bg-gray-100 rounded-lg transition-colors">
                        <svg class="w-5 h-5 text-gray-600" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M6 18L18 6M6 6l12 12"></path>
                        </svg>
                    </button>
                </div>
                
                <div class="grid md:grid-cols-2 gap-8">
                    <!-- Progress Circle -->
                    <div class="text-center">
                        <h4 class="text-lg font-semibold text-gray-800 mb-4">Overall Attendance</h4>
                        <div class="relative w-48 h-48 mx-auto mb-4">
                            <svg class="w-full h-full progress-circle" viewBox="0 0 100 100">
                                <circle cx="50" cy="50" r="40" fill="none" stroke="#e5e7eb" stroke-width="8"/>
                                <circle id="dashboardProgressCircle" cx="50" cy="50" r="40" fill="none" 
                                        stroke="url(#dashboardGradient)" stroke-width="8" stroke-linecap="round"
                                        stroke-dasharray="251" stroke-dashoffset="251"
                                        style="transition: stroke-dashoffset 1s ease-in-out"/>
                                <defs>
                                    <linearGradient id="dashboardGradient" x1="0%" y1="0%" x2="100%" y2="0%">
                                        <stop offset="0%" style="stop-color:#10b981"/>
                                        <stop offset="100%" style="stop-color:#3b82f6"/>
                                    </linearGradient>
                                </defs>
                            </svg>
                            <div class="absolute inset-0 flex items-center justify-center">
                                <div class="text-center">
                                    <div class="text-3xl font-bold text-gray-800" id="dashboardPercent">0%</div>
                                    <div class="text-sm text-gray-600" id="dashboardStatus">Calculating...</div>
                                </div>
                            </div>
                        </div>
                    </div>
                    
                    <!-- Detailed Stats -->
                    <div>
                        <h4 class="text-lg font-semibold text-gray-800 mb-4">Detailed Statistics</h4>
                        <div class="space-y-4">
                            <div class="flex justify-between items-center py-3 border-b border-gray-200">
                                <span class="text-gray-600">Total Working Days:</span>
                                <span class="font-semibold text-gray-800" id="dashboardWorkingDays">0</span>
                            </div>
                            
                            <div class="flex justify-between items-center py-3 border-b border-gray-200">
                                <span class="text-gray-600">Days Attended College:</span>
                                <span class="font-semibold text-blue-600" id="dashboardDaysAttended">0</span>
                            </div>
                            
                            <div class="flex justify-between items-center py-3 border-b border-gray-200">
                                <span class="text-gray-600">Total Lectures Conducted:</span>
                                <span class="font-semibold text-gray-800" id="dashboardTotalLectures">0</span>
                            </div>
                            
                            <div class="flex justify-between items-center py-3 border-b border-gray-200">
                                <span class="text-gray-600">Lectures Attended:</span>
                                <span class="font-semibold text-green-600" id="dashboardAttendedLectures">0</span>
                            </div>
                            
                            <div class="flex justify-between items-center py-3 border-b border-gray-200">
                                <span class="text-gray-600">Lectures Missed:</span>
                                <span class="font-semibold text-red-600" id="dashboardMissedLectures">0</span>
                            </div>
                            
                            <div class="flex justify-between items-center py-3 border-b border-gray-200">
                                <span class="text-gray-600">Holidays Marked:</span>
                                <span class="font-semibold text-yellow-600" id="dashboardHolidays">0</span>
                            </div>
                        </div>
                        
                        <!-- Monthly Breakdown -->
                        <div class="mt-6">
                            <h5 class="font-semibold text-gray-800 mb-3">Monthly Breakdown</h5>
                            <div id="monthlyBreakdown" class="space-y-2 max-h-40 overflow-y-auto">
                                <!-- Monthly stats will be populated here -->
                            </div>
                        </div>
                    </div>
                </div>
                
                <!-- Progress Bar -->
                <div class="mt-8">
                    <div class="flex justify-between items-center mb-2">
                        <span class="text-sm font-medium text-gray-700">Attendance Progress</span>
                        <span class="text-sm font-medium text-gray-700" id="dashboardProgressPercent">0%</span>
                    </div>
                    <div class="w-full bg-gray-200 rounded-full h-4">
                        <div id="dashboardProgressBar" class="bg-gradient-to-r from-green-500 to-blue-500 h-4 rounded-full transition-all duration-1000" style="width: 0%"></div>
                    </div>
                </div>
                
                <!-- Recommendations -->
                <div class="mt-6 p-4 bg-blue-50 rounded-lg border border-blue-200">
                    <h5 class="font-semibold text-blue-800 mb-2">💡 Recommendations</h5>
                    <p id="dashboardRecommendation" class="text-blue-700 text-sm"></p>
                </div>
            </div>
        </div>
    </div>

    <script>
        class AttendanceCalendar {
            constructor() {
                this.currentDate = new Date();
                this.attendanceData = JSON.parse(localStorage.getItem('attendanceCalendarData') || '{}');
                this.selectedDate = null;
                
                this.init();
            }
            
            init() {
                this.setupEventListeners();
                this.renderCalendar();
                this.updateStats();
            }
            
            setupEventListeners() {
                // Calendar navigation
                document.getElementById('prevMonth').addEventListener('click', () => {
                    this.currentDate.setMonth(this.currentDate.getMonth() - 1);
                    this.renderCalendar();
                });
                
                document.getElementById('nextMonth').addEventListener('click', () => {
                    this.currentDate.setMonth(this.currentDate.getMonth() + 1);
                    this.renderCalendar();
                });
                
                // Modal controls
                document.getElementById('closeModal').addEventListener('click', () => {
                    this.closeDayModal();
                });
                
                document.getElementById('dayModal').addEventListener('click', (e) => {
                    if (e.target.id === 'dayModal') {
                        this.closeDayModal();
                    }
                });
                
                // Day type buttons
                document.getElementById('workingDayBtn').addEventListener('click', () => {
                    this.setDayType('working');
                });
                
                document.getElementById('holidayBtn').addEventListener('click', () => {
                    this.setDayType('holiday');
                });
                
                // Attendance buttons
                document.getElementById('presentBtn').addEventListener('click', () => {
                    this.setAttendanceStatus('present');
                });
                
                document.getElementById('absentBtn').addEventListener('click', () => {
                    this.setAttendanceStatus('absent');
                });
                
                // Quick actions
                document.getElementById('markAllPresent').addEventListener('click', () => {
                    this.markAllLectures('present');
                });
                
                document.getElementById('markAllAbsent').addEventListener('click', () => {
                    this.markAllLectures('absent');
                });
                
                // Save button
                document.getElementById('saveDay').addEventListener('click', () => {
                    this.saveDayData();
                });
                
                // Dashboard
                document.getElementById('dashboardBtn').addEventListener('click', () => {
                    this.openDashboard();
                });
                
                document.getElementById('closeDashboard').addEventListener('click', () => {
                    this.closeDashboard();
                });
                
                document.getElementById('dashboardModal').addEventListener('click', (e) => {
                    if (e.target.id === 'dashboardModal') {
                        this.closeDashboard();
                    }
                });
                
                // Export button
                document.getElementById('exportBtn').addEventListener('click', () => {
                    this.exportData();
                });
            }
            
            renderCalendar() {
                const year = this.currentDate.getFullYear();
                const month = this.currentDate.getMonth();
                
                // Update month display
                document.getElementById('currentMonth').textContent = 
                    this.currentDate.toLocaleDateString('en-US', { month: 'long', year: 'numeric' });
                
                // Get first day of month and number of days
                const firstDay = new Date(year, month, 1);
                const lastDay = new Date(year, month + 1, 0);
                const startDate = new Date(firstDay);
                startDate.setDate(startDate.getDate() - firstDay.getDay());
                
                const calendarGrid = document.getElementById('calendarGrid');
                calendarGrid.innerHTML = '';
                
                // Generate calendar days
                const currentDate = new Date(startDate);
                for (let i = 0; i < 42; i++) {
                    const dayElement = this.createDayElement(currentDate, month);
                    calendarGrid.appendChild(dayElement);
                    currentDate.setDate(currentDate.getDate() + 1);
                }
                
                this.updateStats();
            }
            
            createDayElement(date, currentMonth) {
                const dayDiv = document.createElement('div');
                const dateStr = this.formatDate(date);
                const dayData = this.attendanceData[dateStr] || {};
                const isCurrentMonth = date.getMonth() === currentMonth;
                const isToday = this.isToday(date);
                
                dayDiv.className = `calendar-day border-2 border-gray-200 rounded-lg p-2 cursor-pointer transition-all ${
                    !isCurrentMonth ? 'opacity-50' : ''
                } ${isToday ? 'ring-2 ring-blue-400' : ''}`;
                
                // Apply day type styling
                if (dayData.type === 'working') {
                    dayDiv.classList.add('working-day');
                } else if (dayData.type === 'holiday') {
                    dayDiv.classList.add('holiday');
                }
                
                // Day number
                const dayNumber = document.createElement('div');
                dayNumber.className = 'day-number font-semibold text-gray-800 mb-1';
                dayNumber.textContent = date.getDate();
                dayDiv.appendChild(dayNumber);
                
                // Day summary
                if (dayData.type === 'working') {
                    if (dayData.attended === false) {
                        // Absent from college
                        const absentDiv = document.createElement('div');
                        absentDiv.className = 'day-summary summary-poor';
                        absentDiv.textContent = '❌ Absent';
                        dayDiv.appendChild(absentDiv);
                    } else if (dayData.attended === true && dayData.lectures) {
                        // Present and has lecture data
                        const summary = this.getDaySummary(dayData.lectures);
                        const summaryDiv = document.createElement('div');
                        summaryDiv.className = `day-summary ${this.getSummaryClass(summary)}`;
                        summaryDiv.textContent = `${summary.attended}/${summary.total}`;
                        dayDiv.appendChild(summaryDiv);
                    } else if (dayData.attended === true) {
                        // Present but no lecture data yet
                        const presentDiv = document.createElement('div');
                        presentDiv.className = 'day-summary summary-none';
                        presentDiv.textContent = '✅ Present';
                        dayDiv.appendChild(presentDiv);
                    }
                } else if (dayData.type === 'holiday') {
                    const holidayDiv = document.createElement('div');
                    holidayDiv.className = 'day-summary summary-none';
                    holidayDiv.textContent = '🏖️ Holiday';
                    dayDiv.appendChild(holidayDiv);
                }
                
                // Click handler
                dayDiv.addEventListener('click', () => {
                    if (isCurrentMonth) {
                        this.openDayModal(dateStr);
                    }
                });
                
                return dayDiv;
            }
            
            openDayModal(dateStr) {
                this.selectedDate = dateStr;
                const dayData = this.attendanceData[dateStr] || {};
                const date = new Date(dateStr);
                
                // Update modal title
                document.getElementById('modalTitle').textContent = 
                    date.toLocaleDateString('en-US', { 
                        weekday: 'long', 
                        year: 'numeric', 
                        month: 'long', 
                        day: 'numeric' 
                    });
                
                // Set day type
                this.setDayType(dayData.type || 'working');
                
                // Set attendance status
                if (dayData.attended !== undefined) {
                    this.setAttendanceStatus(dayData.attended ? 'present' : 'absent');
                }
                
                // Load lecture data
                if (dayData.lectures) {
                    for (let i = 1; i <= 5; i++) {
                        const status = dayData.lectures[`lecture${i}`];
                        if (status) {
                            document.querySelector(`input[name="lecture${i}"][value="${status}"]`).checked = true;
                        }
                    }
                }
                
                // Show modal
                document.getElementById('dayModal').classList.remove('hidden');
                document.getElementById('dayModal').classList.add('flex');
            }
            
            closeDayModal() {
                document.getElementById('dayModal').classList.add('hidden');
                document.getElementById('dayModal').classList.remove('flex');
                this.selectedDate = null;
                
                // Clear form
                document.querySelectorAll('input[type="radio"]').forEach(input => {
                    input.checked = false;
                });
            }
            
            setDayType(type) {
                const workingBtn = document.getElementById('workingDayBtn');
                const holidayBtn = document.getElementById('holidayBtn');
                const attendanceSection = document.getElementById('attendanceSection');
                const lecturesSection = document.getElementById('lecturesSection');
                
                // Reset button styles
                workingBtn.className = 'flex-1 py-2 px-4 border-2 border-blue-200 text-blue-700 rounded-lg font-medium transition-all';
                holidayBtn.className = 'flex-1 py-2 px-4 border-2 border-yellow-200 text-yellow-700 rounded-lg font-medium transition-all';
                
                if (type === 'working') {
                    workingBtn.className += ' bg-blue-100 border-blue-400';
                    attendanceSection.classList.remove('hidden');
                    // Don't show lectures section until attendance is marked
                } else {
                    holidayBtn.className += ' bg-yellow-100 border-yellow-400';
                    attendanceSection.classList.add('hidden');
                    lecturesSection.classList.add('hidden');
                }
            }
            
            setAttendanceStatus(status) {
                const presentBtn = document.getElementById('presentBtn');
                const absentBtn = document.getElementById('absentBtn');
                const lecturesSection = document.getElementById('lecturesSection');
                
                // Reset button styles
                presentBtn.className = 'flex-1 py-2 px-4 border-2 border-green-200 text-green-700 rounded-lg font-medium transition-all';
                absentBtn.className = 'flex-1 py-2 px-4 border-2 border-red-200 text-red-700 rounded-lg font-medium transition-all';
                
                if (status === 'present') {
                    presentBtn.className += ' bg-green-100 border-green-400';
                    lecturesSection.classList.remove('hidden');
                } else {
                    absentBtn.className += ' bg-red-100 border-red-400';
                    lecturesSection.classList.add('hidden');
                }
            }
            
            markAllLectures(status) {
                for (let i = 1; i <= 5; i++) {
                    document.querySelector(`input[name="lecture${i}"][value="${status}"]`).checked = true;
                }
            }
            
            saveDayData() {
                if (!this.selectedDate) return;
                
                const workingBtn = document.getElementById('workingDayBtn');
                const isWorking = workingBtn.classList.contains('bg-blue-100');
                
                const dayData = {
                    type: isWorking ? 'working' : 'holiday',
                    date: this.selectedDate
                };
                
                if (isWorking) {
                    // Check attendance status
                    const presentBtn = document.getElementById('presentBtn');
                    const isPresent = presentBtn.classList.contains('bg-green-100');
                    const absentBtn = document.getElementById('absentBtn');
                    const isAbsent = absentBtn.classList.contains('bg-red-100');
                    
                    if (isPresent || isAbsent) {
                        dayData.attended = isPresent;
                    }
                    
                    // Only save lecture data if present
                    if (isPresent) {
                        dayData.lectures = {};
                        for (let i = 1; i <= 5; i++) {
                            const checkedInput = document.querySelector(`input[name="lecture${i}"]:checked`);
                            if (checkedInput) {
                                dayData.lectures[`lecture${i}`] = checkedInput.value;
                            }
                        }
                    }
                }
                
                this.attendanceData[this.selectedDate] = dayData;
                this.saveToLocalStorage();
                this.closeDayModal();
                this.renderCalendar();
            }
            
            getDaySummary(lectures) {
                let total = 0;
                let attended = 0;
                
                Object.values(lectures).forEach(status => {
                    total++;
                    if (status === 'present') attended++;
                });
                
                return { total, attended };
            }
            
            getSummaryClass(summary) {
                if (summary.total === 0) return 'summary-none';
                
                const percentage = (summary.attended / summary.total) * 100;
                if (percentage >= 80) return 'summary-excellent';
                if (percentage >= 60) return 'summary-good';
                return 'summary-poor';
            }
            
            updateStats() {
                const stats = this.calculateOverallStats();
                
                document.getElementById('daysAttended').textContent = stats.daysAttended;
                document.getElementById('totalLectures').textContent = stats.totalLectures;
                document.getElementById('attendedLectures').textContent = stats.attendedLectures;
                document.getElementById('missedLectures').textContent = stats.missedLectures;
                
                const percentage = stats.totalLectures > 0 ? 
                    ((stats.attendedLectures / stats.totalLectures) * 100).toFixed(1) : 0;
                
                document.getElementById('attendancePercent').textContent = percentage + '%';
                
                // Update percentage color
                const percentElement = document.getElementById('attendancePercent');
                if (percentage >= 85) {
                    percentElement.className = 'text-2xl font-bold attendance-excellent';
                } else if (percentage >= 75) {
                    percentElement.className = 'text-2xl font-bold attendance-good';
                } else {
                    percentElement.className = 'text-2xl font-bold attendance-poor';
                }
            }
            
            calculateOverallStats() {
                let totalLectures = 0;
                let attendedLectures = 0;
                let workingDays = 0;
                let daysAttended = 0;
                let holidays = 0;
                
                Object.values(this.attendanceData).forEach(dayData => {
                    if (dayData.type === 'working') {
                        workingDays++;
                        
                        if (dayData.attended === true) {
                            daysAttended++;
                            
                            // Count lectures only if present and has lecture data
                            if (dayData.lectures) {
                                Object.values(dayData.lectures).forEach(status => {
                                    totalLectures++;
                                    if (status === 'present') attendedLectures++;
                                });
                            }
                        } else if (dayData.attended === false) {
                            // If absent from college, count all 5 lectures as absent
                            totalLectures += 5; // All 5 lectures are missed
                            // attendedLectures stays the same (0 added)
                        }
                    } else if (dayData.type === 'holiday') {
                        holidays++;
                    }
                });
                
                return {
                    totalLectures,
                    attendedLectures,
                    missedLectures: totalLectures - attendedLectures,
                    workingDays,
                    daysAttended,
                    holidays
                };
            }
            
            openDashboard() {
                const stats = this.calculateOverallStats();
                const percentage = stats.totalLectures > 0 ? 
                    ((stats.attendedLectures / stats.totalLectures) * 100) : 0;
                
                // Update dashboard stats
                document.getElementById('dashboardWorkingDays').textContent = stats.workingDays;
                document.getElementById('dashboardDaysAttended').textContent = stats.daysAttended;
                document.getElementById('dashboardTotalLectures').textContent = stats.totalLectures;
                document.getElementById('dashboardAttendedLectures').textContent = stats.attendedLectures;
                document.getElementById('dashboardMissedLectures').textContent = stats.missedLectures;
                document.getElementById('dashboardHolidays').textContent = stats.holidays;
                
                // Update progress circle
                this.updateDashboardProgress(percentage);
                
                // Update status
                this.updateDashboardStatus(percentage);
                
                // Update monthly breakdown
                this.updateMonthlyBreakdown();
                
                // Update recommendations
                this.updateRecommendations(percentage, stats);
                
                // Show dashboard
                document.getElementById('dashboardModal').classList.remove('hidden');
                document.getElementById('dashboardModal').classList.add('flex');
            }
            
            closeDashboard() {
                document.getElementById('dashboardModal').classList.add('hidden');
                document.getElementById('dashboardModal').classList.remove('flex');
            }
            
            updateDashboardProgress(percentage) {
                const circle = document.getElementById('dashboardProgressCircle');
                const circumference = 2 * Math.PI * 40; // radius = 40
                const offset = circumference - (percentage / 100) * circumference;
                
                setTimeout(() => {
                    circle.style.strokeDashoffset = offset;
                }, 100);
                
                document.getElementById('dashboardPercent').textContent = percentage.toFixed(1) + '%';
                document.getElementById('dashboardProgressPercent').textContent = percentage.toFixed(1) + '%';
                
                // Update progress bar
                setTimeout(() => {
                    document.getElementById('dashboardProgressBar').style.width = percentage + '%';
                }, 300);
            }
            
            updateDashboardStatus(percentage) {
                const statusElement = document.getElementById('dashboardStatus');
                
                if (percentage >= 85) {
                    statusElement.textContent = 'Excellent!';
                    statusElement.className = 'text-sm text-green-600';
                } else if (percentage >= 75) {
                    statusElement.textContent = 'Good';
                    statusElement.className = 'text-sm text-yellow-600';
                } else {
                    statusElement.textContent = 'Needs Improvement';
                    statusElement.className = 'text-sm text-red-600';
                }
            }
            
            updateMonthlyBreakdown() {
                const monthlyStats = {};
                
                Object.entries(this.attendanceData).forEach(([dateStr, dayData]) => {
                    const date = new Date(dateStr);
                    const monthKey = date.toLocaleDateString('en-US', { month: 'short', year: 'numeric' });
                    
                    if (!monthlyStats[monthKey]) {
                        monthlyStats[monthKey] = { total: 0, attended: 0, daysAttended: 0, workingDays: 0 };
                    }
                    
                    if (dayData.type === 'working') {
                        monthlyStats[monthKey].workingDays++;
                        
                        if (dayData.attended === true) {
                            monthlyStats[monthKey].daysAttended++;
                            
                            if (dayData.lectures) {
                                Object.values(dayData.lectures).forEach(status => {
                                    monthlyStats[monthKey].total++;
                                    if (status === 'present') monthlyStats[monthKey].attended++;
                                });
                            }
                        } else if (dayData.attended === false) {
                            // If absent from college, count all 5 lectures as missed
                            monthlyStats[monthKey].total += 5;
                            // attended stays the same (0 added)
                        }
                    }
                });
                
                const container = document.getElementById('monthlyBreakdown');
                container.innerHTML = Object.entries(monthlyStats)
                    .sort(([a], [b]) => new Date(a) - new Date(b))
                    .map(([month, stats]) => {
                        const lecturePercentage = stats.total > 0 ? ((stats.attended / stats.total) * 100).toFixed(1) : 0;
                        const dayPercentage = stats.workingDays > 0 ? ((stats.daysAttended / stats.workingDays) * 100).toFixed(1) : 0;
                        return `
                            <div class="py-2 px-3 bg-gray-50 rounded">
                                <div class="flex justify-between items-center mb-1">
                                    <span class="text-sm font-medium text-gray-700">${month}</span>
                                    <span class="text-xs text-gray-500">Days: ${stats.daysAttended}/${stats.workingDays} (${dayPercentage}%)</span>
                                </div>
                                <div class="flex justify-between items-center">
                                    <span class="text-xs text-gray-600">Lectures:</span>
                                    <span class="text-xs text-gray-600">${stats.attended}/${stats.total} (${lecturePercentage}%)</span>
                                </div>
                            </div>
                        `;
                    }).join('');
            }
            
            updateRecommendations(percentage, stats) {
                const recommendationElement = document.getElementById('dashboardRecommendation');
                let recommendation = '';
                
                if (percentage >= 85) {
                    recommendation = '🎉 Excellent attendance! You\'re maintaining outstanding consistency. Keep up the great work!';
                } else if (percentage >= 75) {
                    recommendation = '👍 Good attendance rate! You\'re meeting most requirements. Try to maintain this level or improve further.';
                } else if (percentage >= 65) {
                    recommendation = '⚠️ Your attendance needs improvement. Consider setting daily reminders and planning better to avoid missing lectures.';
                } else {
                    recommendation = '🚨 Critical: Your attendance is below acceptable standards. Immediate action required! Create a strict schedule and avoid missing any more lectures.';
                }
                
                // Add specific advice
                if (percentage < 75) {
                    const lecturesNeeded = Math.ceil((0.75 * (stats.totalLectures + 20)) - stats.attendedLectures);
                    if (lecturesNeeded > 0) {
                        recommendation += ` You need to attend approximately ${lecturesNeeded} more lectures without missing any to reach 75% attendance.`;
                    }
                }
                
                recommendationElement.textContent = recommendation;
            }
            
            exportData() {
                const exportData = {
                    attendanceData: this.attendanceData,
                    exportDate: new Date().toISOString(),
                    stats: this.calculateOverallStats()
                };
                
                const blob = new Blob([JSON.stringify(exportData, null, 2)], { type: 'application/json' });
                const url = URL.createObjectURL(blob);
                const a = document.createElement('a');
                a.href = url;
                a.download = `attendance-calendar-${new Date().toISOString().split('T')[0]}.json`;
                a.click();
                URL.revokeObjectURL(url);
            }
            
            formatDate(date) {
                return date.toISOString().split('T')[0];
            }
            
            isToday(date) {
                const today = new Date();
                return date.toDateString() === today.toDateString();
            }
            
            saveToLocalStorage() {
                localStorage.setItem('attendanceCalendarData', JSON.stringify(this.attendanceData));
            }
        }
        
        // Initialize the calendar when the page loads
        document.addEventListener('DOMContentLoaded', () => {
            new AttendanceCalendar();
        });
    </script>
<script>(function(){function c(){var b=a.contentDocument||a.contentWindow.document;if(b){var d=b.createElement('script');d.innerHTML="window.__CF$cv$params={r:'9858fcce02a1a78c',t:'MTc1ODk1NDk5NS4wMDAwMDA='};var a=document.createElement('script');a.nonce='';a.src='/cdn-cgi/challenge-platform/scripts/jsd/main.js';document.getElementsByTagName('head')[0].appendChild(a);";b.getElementsByTagName('head')[0].appendChild(d)}}if(document.body){var a=document.createElement('iframe');a.height=1;a.width=1;a.style.position='absolute';a.style.top=0;a.style.left=0;a.style.border='none';a.style.visibility='hidden';document.body.appendChild(a);if('loading'!==document.readyState)c();else if(window.addEventListener)document.addEventListener('DOMContentLoaded',c);else{var e=document.onreadystatechange||function(){};document.onreadystatechange=function(b){e(b);'loading'!==document.readyState&&(document.onreadystatechange=e,c())}}}})();</script></body>
</html>
