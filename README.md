<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>PO Management System Dashboard</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css">
    <link href="https://fonts.googleapis.com/css2?family=Inter:wght@300;400;600;700&display=swap" rel="stylesheet">
    <style>
        /* Custom CSS Variables for Theming */
        :root {
            --primary-color: #3b82f6; /* blue-500 */
            --primary-dark-color: #2563eb; /* blue-600 */
            --primary-text-color: #1e40af; /* blue-800 */
            --accent-color: #6366f1; /* indigo-500 */
            --accent-dark-color: #4f46e5; /* indigo-600 */
            --accent-text-color: #4338ca; /* indigo-800 */
            --border-color: #bfdbfe; /* blue-200 */
        }

        /* Apply Inter font globally */
        body {
            font-family: 'Inter', sans-serif;
            background-color: #f3f4f6; /* Light gray background */
        }

        /* Custom Toast Notification Styles */
        #toast-container {
            position: fixed;
            top: 20px;
            right: 20px;
            z-index: 1000;
            display: flex;
            flex-direction: column;
            gap: 10px;
        }
        .toast {
            background-color: #333;
            color: white;
            padding: 10px 20px;
            border-radius: 8px;
            box-shadow: 0 4px 12px rgba(0,0,0,0.15);
            opacity: 0;
            transition: opacity 0.3s ease-in-out, transform 0.3s ease-in-out;
            transform: translateY(-20px);
            min-width: 250px;
            display: flex;
            align-items: center;
            gap: 10px;
        }
        .toast.show {
            opacity: 1;
            transform: translateY(0);
        }
        .toast.success { background-color: #22c55e; } /* green-500 */
        .toast.error { background-color: #ef4444; }   /* red-500 */
        .toast.info { background-color: #3b82f6; }    /* blue-500 */

        /* Table sorting indicators */
        .sortable-header {
            cursor: pointer;
            position: relative;
            padding-right: 20px; /* Space for icon */
        }
        .sortable-header .sort-icon {
            position: absolute;
            right: 5px;
            top: 50%;
            transform: translateY(-50%);
            font-size: 0.8em;
            color: #aaa;
        }
        .sortable-header.asc .sort-icon::before { content: '▲'; color: var(--primary-color); }
        .sortable-header.desc .sort-icon::before { content: '▼'; color: var(--primary-color); }
        .sortable-header:not(.asc):not(.desc) .sort-icon::before { content: '↕'; }

        /* Dynamic button styles using CSS variables */
        .btn-primary {
            background-color: var(--primary-color);
            color: white;
            font-weight: 600;
            padding: 0.75rem 1.5rem;
            border-radius: 0.75rem;
            transition: background-color 0.2s ease-in-out;
        }
        .btn-primary:hover {
            background-color: var(--primary-dark-color);
        }

        .btn-accent {
            background-color: var(--accent-color);
            color: white;
            font-weight: 600;
            padding: 0.75rem 1.5rem;
            border-radius: 0.75rem;
            transition: background-color 0.2s ease-in-out;
        }
        .btn-accent:hover {
            background-color: var(--accent-dark-color);
        }

        /* Dynamic border colors for modals and sections */
        .border-t-theme-primary {
            border-top-color: var(--primary-color);
        }
        .border-theme-primary-light {
            border-color: var(--border-color);
        }
        .text-theme-primary {
            color: var(--primary-text-color);
        }
        .text-theme-accent {
            color: var(--accent-text-color);
        }
    </style>
</head>
<body class="bg-gray-100">
    <div class="container mx-auto p-6">

        <!-- Toast Notification Container -->
        <div id="toast-container"></div>

        <!-- Login Modal -->
        <div id="loginModal" class="fixed inset-0 bg-blue-900/80 backdrop-blur-md flex items-center justify-center z-50">
            <div class="bg-white p-8 rounded-2xl shadow-2xl max-w-md w-full border-t-8 border-theme-primary">
                <div class="text-center mb-6">
                    <h2 class="text-3xl font-bold text-theme-primary">
                        <i class="fas fa-user-tie mr-2"></i>
                        Department Login
                    </h2>
                </div>
                <form id="loginForm" class="space-y-6">
                    <div>
                        <label for="department" class="block font-bold mb-2 text-gray-700">Department</label>
                        <select id="department" class="w-full border border-gray-300 rounded-lg p-3 focus:ring-2 focus:ring-blue-400 focus:border-transparent transition duration-200" required>
                            <option value="">-- Select Department --</option>
                            <option value="spinning">Spinning</option>
                            <option value="knitting">Knitting</option>
                            <option value="process">Process</option>
                            <option value="garments">Garments</option>
                            <option value="admin">Admin</option>
                        </select>
                    </div>
                    <div>
                        <label for="password" class="block font-bold mb-2 text-gray-700">Password</label>
                        <input type="password" id="password" placeholder="Password (e.g., 12345 or sunflag12345 for Admin)" class="w-full border border-gray-300 rounded-lg p-3 focus:ring-2 focus:ring-blue-400 focus:border-transparent transition duration-200" required autocomplete="current-password">
                    </div>
                    <button type="submit" class="w-full btn-primary">Login</button>
                    <div class="text-center mt-4">
                        <button type="button" id="changePasswordBtn" class="text-blue-500 hover:text-blue-700 underline font-semibold transition duration-200">Change Password</button>
                    </div>
                </form>
            </div>
        </div>

        <!-- Change Password Modal -->
        <div id="changePasswordModal" class="fixed inset-0 bg-gray-900/50 backdrop-blur-md flex items-center justify-center z-50 hidden">
            <div class="bg-white p-8 rounded-2xl shadow-2xl max-w-md w-full border-t-8 border-theme-accent">
                <div class="text-center mb-6">
                    <h2 class="text-2xl font-bold text-theme-accent"><i class="fas fa-key mr-2"></i>Change Password</h2>
                </div>
                <form id="changePasswordForm" class="space-y-4">
                    <div>
                        <label for="deptEmail" class="block font-bold mb-2 text-gray-700">Department Email Address</label>
                        <input type="email" id="deptEmail" class="w-full border border-gray-300 rounded-lg p-3 focus:ring-2 focus:ring-purple-400 focus:border-transparent transition duration-200" placeholder="your.department@email.com" required>
                    </div>
                    <div>
                        <label for="newPassword" class="block font-bold mb-2 text-gray-700">New Password</label>
                        <input type="password" id="newPassword" class="w-full border border-gray-300 rounded-lg p-3 focus:ring-2 focus:ring-purple-400 focus:border-transparent transition duration-200" placeholder="Enter new password" required>
                    </div>
                    <button type="submit" class="w-full btn-accent">Request Change</button>
                    <div class="text-center mt-2">
                        <button type="button" id="cancelChangePassword" class="text-gray-500 hover:text-gray-700 underline transition duration-200">Cancel</button>
                    </div>
                </form>
            </div>
        </div>

        <!-- Main Content -->
        <div id="mainContent" class="hidden">
            <div class="mb-4 flex justify-between items-center">
                <h1 class="text-4xl font-bold text-theme-primary mb-2">PO Dashboard (<span id="deptName"></span> Department)</h1>
                <div class="flex items-center space-x-2">
                    <button id="reportBtn" class="btn-accent">See Reports</button>
                    <button id="downloadExcelBtn" class="bg-green-600 hover:bg-green-700 text-white font-bold py-2 px-4 rounded-lg transition duration-200">
                        <i class="fas fa-file-excel mr-2"></i>Download Excel
                    </button>
                    <button id="logoutBtn" class="bg-gray-400 hover:bg-gray-500 text-white font-bold py-2 px-4 rounded-lg transition duration-200">Logout</button>
                </div>
            </div>

            <!-- Daily Data Entry Section (Hidden for Admin) -->
            <div id="dailyEntrySection" class="mb-8 bg-white rounded-xl shadow-lg p-6 border border-theme-primary-light">
                <h2 class="text-2xl font-bold mb-4 text-theme-accent">Daily PO Entry</h2>
                <form id="dailyEntryForm" class="grid grid-cols-1 md:grid-cols-2 gap-4">
                    <input required class="border border-gray-300 rounded-lg p-3 focus:ring-2 focus:ring-blue-400 focus:border-transparent transition duration-200" name="poNo" placeholder="PO No">
                    <input required class="border border-gray-300 rounded-lg p-3 focus:ring-2 focus:ring-blue-400 focus:border-transparent transition duration-200" name="date" type="date" placeholder="Date">
                    <input required class="border border-gray-300 rounded-lg p-3 focus:ring-2 focus:ring-blue-400 focus:border-transparent transition duration-200" name="lotNo" placeholder="Lot No">
                    <input required class="border border-gray-300 rounded-lg p-3 focus:ring-2 focus:ring-blue-400 focus:border-transparent transition duration-200" name="dia" placeholder="DIA">
                    <input required class="border border-gray-300 rounded-lg p-3 focus:ring-2 focus:ring-blue-400 focus:border-transparent transition duration-200" name="fabricKg" type="number" min="1" placeholder="Fabric KG" title="Fabric KG must be a positive number">
                    <input required class="border border-gray-300 rounded-lg p-3 focus:ring-2 focus:ring-blue-400 focus:border-transparent transition duration-200" name="customer" placeholder="Customer Name">
                    <input required class="border border-gray-300 rounded-lg p-3 focus:ring-2 focus:ring-blue-400 focus:border-transparent transition duration-200" name="quality" placeholder="Quality">
                    <input required class="border border-gray-300 rounded-lg p-3 focus:ring-2 focus:ring-blue-400 focus:border-transparent transition duration-200" name="colour" placeholder="Colour">
                    <button class="col-span-1 md:col-span-2 bg-green-500 hover:bg-green-600 text-white font-bold py-3 rounded-xl transition duration-200" type="submit">Add PO Entry</button>
                </form>
            </div>

            <!-- Admin Tools Section (Visible only for Admin) -->
            <div id="adminToolsSection" class="mb-8 bg-white rounded-xl shadow-lg p-6 border border-orange-200 hidden">
                <h2 class="text-2xl font-bold mb-4 text-orange-600"><i class="fas fa-user-shield mr-2"></i>Admin Tools</h2>
                <h3 class="text-xl font-semibold mb-3 text-orange-500">Pending Password Change Requests</h3>
                <div id="passwordRequestsList" class="space-y-3">
                    <!-- Password change requests will be rendered here -->
                    <p class="text-gray-500" id="noRequestsMessage">No pending requests.</p>
                </div>
            </div>

            <!-- PO List Table -->
            <div class="bg-white p-6 rounded-xl shadow-lg border border-theme-primary-light mb-8">
                <h2 class="text-2xl font-bold mb-4 text-theme-accent">POs & Stages</h2>
                <!-- Filters Section -->
                <div class="mb-4 flex flex-wrap gap-4 items-center">
                    <div id="stageFilterContainer">
                        <label for="stageFilter" class="block text-sm font-medium text-gray-700">Filter by Stage:</label>
                        <select id="stageFilter" class="mt-1 block w-full pl-3 pr-10 py-2 text-base border border-gray-300 rounded-lg focus:outline-none focus:ring-2 focus:ring-blue-400 focus:border-transparent sm:text-sm transition duration-200"></select>
                    </div>
                    <div>
                        <label for="diaFilter" class="block text-sm font-medium text-gray-700">Filter by DIA:</label>
                        <input type="text" id="diaFilter" placeholder="Enter DIA" class="mt-1 block w-full pl-3 pr-3 py-2 border border-gray-300 rounded-lg focus:outline-none focus:ring-2 focus:ring-blue-400 focus:border-transparent sm:text-sm transition duration-200">
                    </div>
                    <div>
                        <label for="customerFilter" class="block text-sm font-medium text-gray-700">Filter by Customer:</label>
                        <input type="text" id="customerFilter" placeholder="Enter Customer Name" class="mt-1 block w-full pl-3 pr-3 py-2 border border-gray-300 rounded-lg focus:outline-none focus:ring-2 focus:ring-blue-400 focus:border-transparent sm:text-sm transition duration-200">
                    </div>
                    <div id="departmentFilterContainer" class="hidden">
                        <label for="departmentFilter" class="block text-sm font-medium text-gray-700">Filter by Department:</label>
                        <select id="departmentFilter" class="mt-1 block w-full pl-3 pr-10 py-2 text-base border border-gray-300 rounded-lg focus:outline-none focus:ring-2 focus:ring-blue-400 focus:border-transparent sm:text-sm transition duration-200">
                            <option value="">All Departments</option>
                            <option value="spinning">Spinning</option>
                            <option value="knitting">Knitting</option>
                            <option value="process">Process</option>
                            <option value="garments">Garments</option>
                        </select>
                    </div>
                    <button id="applyFilters" class="mt-auto btn-primary">Apply Filters</button>
                </div>
                <div class="overflow-x-auto">
                    <table class="min-w-full border-collapse rounded-lg overflow-hidden text-xs md:text-base bg-white">
                        <thead>
                            <tr class="bg-gray-100 text-gray-700 uppercase text-sm leading-normal">
                                <th class="py-3 px-6 text-left border-b border-gray-200 sortable-header" data-sort="poNo">PO No <span class="sort-icon"></span></th>
                                <th class="py-3 px-6 text-left border-b border-gray-200 sortable-header" data-sort="date">Date <span class="sort-icon"></span></th>
                                <th class="py-3 px-6 text-left border-b border-gray-200">Lot No</th>
                                <th class="py-3 px-6 text-left border-b border-gray-200">DIA</th>
                                <th class="py-3 px-6 text-left border-b border-gray-200 sortable-header" data-sort="fabricKg">Fabric KG <span class="sort-icon"></span></th>
                                <th class="py-3 px-6 text-left border-b border-gray-200">Customer</th>
                                <th class="py-3 px-6 text-left border-b border-gray-200">Quality</th>
                                <th class="py-3 px-6 text-left border-b border-gray-200">Colour</th>
                                <th class="py-3 px-6 text-left border-b border-gray-200 department-header">Department</th>
                                <th class="py-3 px-6 text-left border-b border-gray-200 sortable-header" data-sort="stage">Current Stage <span class="sort-icon"></span></th>
                                <th class="py-3 px-6 text-left border-b border-gray-200">History</th>
                                <th class="py-3 px-6 text-left border-b border-gray-200">Actions</th>
                            </tr>
                        </thead>
                        <tbody id="poList" class="text-gray-600 text-sm font-light">
                            <!-- PO rows will be rendered here -->
                        </tbody>
                    </table>
                </div>
            </div>
        </div>

        <!-- Report Modal -->
        <div id="reportModal" class="fixed inset-0 bg-black/40 backdrop-blur-md flex items-center justify-center z-50 hidden">
            <div class="bg-white p-8 rounded-2xl shadow-2xl max-w-3xl w-full border-t-8 border-teal-400">
                <div class="flex justify-between items-center mb-6">
                    <h2 class="text-2xl font-bold text-teal-800"><i class="fas fa-chart-bar mr-2"></i>Department Reports</h2>
                    <button id="closeReport" class="text-gray-500 hover:text-black text-lg transition duration-200"><i class="fas fa-times"></i></button>
                </div>
                <div id="reportContent">
                    <canvas id="poGraph" height="120"></canvas>
                    <ul class="mt-6 space-y-2 text-gray-700 text-base">
                        <li><b>Total PO Received:</b> <span id="totalReceived"></span></li>
                        <li><b>Total Production:</b> <span id="totalProduction"></span></li>
                        <li><b>Total Completed & Sent:</b> <span id="totalCompleted"></span></li>
                        <li><b>PO Balance to Deliver (DIA-wise):</b> <span id="poBalance"></span></li>
                        <li><b>Estimated Days Remaining for Balance Delivery:</b> <span id="daysRemain"></span></li>
                    </ul>
                </div>
            </div>
        </div>

        <!-- Change PO Stage Modal -->
        <div id="changePoStageModal" class="fixed inset-0 bg-gray-900/50 backdrop-blur-md flex items-center justify-center z-50 hidden">
            <div class="bg-white p-8 rounded-2xl shadow-2xl max-w-md w-full border-t-8 border-theme-primary">
                <div class="text-center mb-6">
                    <h2 class="text-2xl font-bold text-theme-primary"><i class="fas fa-exchange-alt mr-2"></i>Change PO Stage</h2>
                    <p class="text-gray-600">PO No: <span id="modalPoNo" class="font-bold"></span></p>
                </div>
                <form id="changePoStageForm" class="space-y-4">
                    <input type="hidden" id="hiddenPoNo">
                    <input type="hidden" id="hiddenPoDept">
                    <div>
                        <label for="newPoStage" class="block font-bold mb-2 text-gray-700">Select New Stage</label>
                        <select id="newPoStage" class="w-full border border-gray-300 rounded-lg p-3 focus:ring-2 focus:ring-blue-400 focus:border-transparent transition duration-200" required>
                            <!-- Options will be populated by JavaScript -->
                        </select>
                    </div>
                    <div>
                        <label for="stageNote" class="block font-bold mb-2 text-gray-700">Note (Optional)</label>
                        <input type="text" id="stageNote" class="w-full border border-gray-300 rounded-lg p-3 focus:ring-2 focus:ring-blue-400 focus:border-transparent transition duration-200" placeholder="Add a note for this stage change">
                    </div>
                    <button type="submit" class="w-full btn-primary">Update Stage</button>
                    <div class="text-center mt-2">
                        <button type="button" id="cancelChangePoStage" class="text-gray-500 hover:text-gray-700 underline transition duration-200">Cancel</button>
                    </div>
                </form>
            </div>
        </div>

        <!-- PO History Modal -->
        <div id="poHistoryModal" class="fixed inset-0 bg-gray-900/50 backdrop-blur-md flex items-center justify-center z-50 hidden">
            <div class="bg-white p-8 rounded-2xl shadow-2xl max-w-xl w-full border-t-8 border-theme-accent">
                <div class="flex justify-between items-center mb-6">
                    <h2 class="text-2xl font-bold text-theme-accent"><i class="fas fa-history mr-2"></i>PO History for <span id="historyModalPoNo" class="font-bold"></span></h2>
                    <button id="closeHistoryModal" class="text-gray-500 hover:text-black text-lg transition duration-200"><i class="fas fa-times"></i></button>
                </div>
                <div id="historyContent" class="max-h-96 overflow-y-auto space-y-4 pr-2">
                    <!-- History details will be rendered here -->
                </div>
            </div>
        </div>

    </div>
    <script>
    // Department stage maps
    const stagesMap = {
        spinning: ["Cotton Receipt", "Carding", "Drawing", "Roving", "Spinning", "Yarn Packing", "Completed"],
        knitting: ["Yarn Receipt", "Knitting", "Inspection", "Packing", "Completed"],
        process: ["Godown Receipt", "Dyeing", "Balloon Machine", "TLD Machine", "Quality Checking", "Shipped", "Completed"],
        garments: ["Design", "Cutting", "Sewing", "Finishing", "Packing", "Delivery", "Completed"]
    };

    // Department specific themes (Tailwind CSS classes or custom CSS variables)
    const departmentThemes = {
        spinning: {
            primary: '#3b82f6', // blue-500
            primaryDark: '#2563eb', // blue-600
            primaryText: '#1e40af', // blue-800
            accent: '#10b981', // emerald-500
            accentDark: '#059669', // emerald-600
            accentText: '#065f46', // emerald-800
            border: '#bfdbfe' // blue-200
        },
        knitting: {
            primary: '#ef4444', // red-500
            primaryDark: '#dc2626', // red-600
            primaryText: '#991b1b', // red-800
            accent: '#f59e0b', // amber-500
            accentDark: '#d97706', // amber-600
            accentText: '#b45309', // amber-800
            border: '#fecaca' // red-200
        },
        process: {
            primary: '#8b5cf6', // violet-500
            primaryDark: '#7c3aed', // violet-600
            primaryText: '#5b21b6', // violet-800
            accent: '#ec4899', // pink-500
            accentDark: '#db2777', // pink-600
            accentText: '#9d174d', // pink-800
            border: '#ddd6fe' // violet-200
        },
        garments: {
            primary: '#06b6d4', // cyan-500
            primaryDark: '#0891b2', // cyan-600
            primaryText: '#164e63', // cyan-800
            accent: '#65a30d', // lime-600
            accentDark: '#4d7c0f', // lime-700
            accentText: '#3f6212', // lime-800
            border: '#a5f3fc' // cyan-200
        },
        admin: {
            primary: '#f97316', // orange-500
            primaryDark: '#ea580c', // orange-600
            primaryText: '#9a3412', // orange-800
            accent: '#6b7280', // gray-500
            accentDark: '#4b5563', // gray-600
            accentText: '#374151', // gray-800
            border: '#fed7aa' // orange-200
        }
    };

    function applyDepartmentTheme(department) {
        const theme = departmentThemes[department] || departmentThemes.admin; // Fallback to admin theme
        const root = document.documentElement;
        root.style.setProperty('--primary-color', theme.primary);
        root.style.setProperty('--primary-dark-color', theme.primaryDark);
        root.style.setProperty('--primary-text-color', theme.primaryText);
        root.style.setProperty('--accent-color', theme.accent);
        root.style.setProperty('--accent-dark-color', theme.accentDark);
        root.style.setProperty('--accent-text-color', theme.accentText);
        root.style.setProperty('--border-color', theme.border);

        // Update specific elements that use direct Tailwind classes if needed
        // For example, the admin tools section border
        const adminToolsSection = document.getElementById('adminToolsSection');
        if (adminToolsSection) {
            adminToolsSection.style.borderColor = theme.border;
        }
    }

    // --- Local Storage Management ---
    function saveToLocalStorage(key, data) {
        try {
            localStorage.setItem(key, JSON.stringify(data));
        } catch (e) {
            showToast('Error saving data to local storage.', 'error');
            console.error('Error saving to local storage:', e);
        }
    }

    function loadFromLocalStorage(key, defaultValue) {
        try {
            const data = localStorage.getItem(key);
            return data ? JSON.parse(data) : defaultValue;
        } catch (e) {
            showToast('Error loading data from local storage. Using default.', 'error');
            console.error('Error loading from local storage:', e);
            return defaultValue;
        }
    }

    // Initial data load from local storage or default
    let pos = loadFromLocalStorage('po_data', [
        {
            poNo: 'PO1001', date: '2025-07-10', lotNo: 'L1', dia: 'DIA123', fabricKg: 100,
            customer: 'ABC', quality: 'Premium', colour: 'Red', department: 'spinning', stage: 'Cotton Receipt',
            history: [{stage: 'Cotton Receipt', date: '2025-07-10', user: 'spinningmgr1', note: 'Cotton received'}]
        },
        {
            poNo: 'PO1002', date: '2025-07-11', lotNo: 'L2', dia: 'DIA124', fabricKg: 200,
            customer: 'XYZ', quality: 'Standard', colour: 'Blue', department: 'knitting', stage: 'Knitting',
            history: [
                {stage: 'Yarn Receipt', date: '2025-07-11', user: 'knittingmgr1', note: 'Yarn ok'},
                {stage: 'Knitting', date: '2025-07-12', user: 'knittingmgr1', note: 'Knitting started'}
            ]
        },
        {
            poNo: 'PO1003', date: '2025-07-09', lotNo: 'L3', dia: 'DIA125', fabricKg: 150,
            customer: 'DEF', quality: 'Premium', colour: 'Black', department: 'process', stage: 'Dyeing',
            history: [
                {stage: 'Godown Receipt', date: '2025-07-09', user: 'processmgr1', note: 'Received'},
                {stage: 'Dyeing', date: '2025-07-10', user: 'dyer1', note: 'Dye started'}
            ]
        },
        {
            poNo: 'PO1004', date: '2025-07-08', lotNo: 'L4', dia: 'DIA126', fabricKg: 120,
            customer: 'LMN', quality: 'Standard', colour: 'Green', department: 'garments', stage: 'Cutting',
            history: [
                {stage: 'Design', date: '2025-07-08', user: 'garmentsmgr1', note: 'Design ok'},
                {stage: 'Cutting', date: '2025-07-09', user: 'garmentsmgr1', note: 'Cutting started'}
            ]
        }
    ]);

    let currentDepartment = '';
    let LOGIN_PASSWORDS = loadFromLocalStorage('login_passwords', {
        spinning: '12345',
        knitting: '12345',
        process: '12345',
        garments: '12345',
        admin: 'sunflag12345'
    });

    let pendingPasswordRequests = loadFromLocalStorage('pending_password_requests', []);

    // --- Toast Notification Function ---
    function showToast(message, type = 'info', duration = 3000) {
        const toastContainer = document.getElementById('toast-container');
        const toast = document.createElement('div');
        toast.className = `toast ${type}`;
        toast.innerHTML = `<i class="fas ${type === 'success' ? 'fa-check-circle' : type === 'error' ? 'fa-times-circle' : 'fa-info-circle'} mr-2"></i> ${message}`;
        toastContainer.appendChild(toast);

        // Trigger reflow to ensure transition plays
        void toast.offsetWidth;

        toast.classList.add('show');

        setTimeout(() => {
            toast.classList.remove('show');
            toast.addEventListener('transitionend', () => toast.remove(), { once: true });
        }, duration);
    }

    function getNextStage(dept, stage) {
        const stages = stagesMap[dept];
        const idx = stages.indexOf(stage);
        if (idx >= 0 && idx < stages.length - 1) {
            return stages[idx + 1];
        }
        return '';
    }

    function renderStageFilter(dept) {
        const stageFilter = document.getElementById('stageFilter');
        stageFilter.innerHTML = `<option value="">All Stages</option>`;
        if (dept !== 'admin') {
            stagesMap[dept].forEach(s => {
                stageFilter.innerHTML += `<option value="${s}">${s}</option>`;
            });
        } else {
            const allStages = new Set();
            Object.values(stagesMap).forEach(deptStages => {
                deptStages.forEach(stage => allStages.add(stage));
            });
            Array.from(allStages).sort().forEach(s => {
                stageFilter.innerHTML += `<option value="${s}">${s}</option>`;
            });
        }
    }

    let currentSortColumn = null;
    let currentSortDirection = 'asc'; // 'asc' or 'desc'

    function sortPOs(column, direction) {
        pos.sort((a, b) => {
            let valA = a[column];
            let valB = b[column];

            // Handle numeric sorting for fabricKg
            if (column === 'fabricKg') {
                valA = Number(valA);
                valB = Number(valB);
            }
            // Handle date sorting
            if (column === 'date') {
                valA = new Date(valA);
                valB = new Date(valB);
            }

            if (valA < valB) return direction === 'asc' ? -1 : 1;
            if (valA > valB) return direction === 'asc' ? 1 : -1;
            return 0;
        });
        renderPOs(
            document.getElementById('stageFilter').value,
            document.getElementById('diaFilter').value.trim(),
            document.getElementById('customerFilter').value.trim(),
            document.getElementById('departmentFilter').value
        );
    }

    function renderPOs(stageFilter = '', diaFilter = '', customerFilter = '', departmentFilter = '') {
        const poList = document.getElementById('poList');
        poList.innerHTML = '';

        const is_admin = currentDepartment === 'admin';
        const departmentHeaderTh = document.querySelector('th.department-header');

        // Show/hide Department column header
        if (is_admin) {
            departmentHeaderTh.style.display = 'table-cell'; // Show
        } else {
            departmentHeaderTh.style.display = 'none'; // Hide
        }

        // Update sort icons
        document.querySelectorAll('.sortable-header').forEach(header => {
            header.classList.remove('asc', 'desc');
            if (header.dataset.sort === currentSortColumn) {
                header.classList.add(currentSortDirection);
            }
        });

        let filteredPOs = pos.filter(po => {
            const matchesDepartment = is_admin ? (!departmentFilter || po.department === departmentFilter) : (po.department === currentDepartment);
            const matchesStage = (!stageFilter || po.stage === stageFilter);
            const matchesDia = (!diaFilter || po.dia.toLowerCase().includes(diaFilter.toLowerCase()));
            const matchesCustomer = (!customerFilter || po.customer.toLowerCase().includes(customerFilter.toLowerCase()));

            return matchesDepartment && matchesStage && matchesDia && matchesCustomer;
        });

        filteredPOs.forEach(po => {
            let actions = '';
            if (po.stage !== "Completed" || is_admin) {
                actions += `<button class="update-stage-btn bg-green-500 hover:bg-green-600 text-white px-2 py-1 rounded-md text-xs mb-1 w-full transition duration-200" data-po="${po.poNo}" data-dept="${po.department}">
                    <i class="fas fa-arrow-right"></i> Next Stage
                </button>`;
                actions += `<button class="change-stage-btn bg-blue-500 hover:bg-blue-600 text-white px-2 py-1 rounded-md text-xs mb-1 w-full transition duration-200" data-po="${po.poNo}" data-dept="${po.department}">
                    <i class="fas fa-exchange-alt"></i> Change Stage
                </button>`;
            } else {
                actions = `<span class="text-green-700 font-bold text-xs">✓ Completed</span>`;
            }
            actions += `<button class="delete-po-btn bg-red-500 hover:bg-red-600 text-white px-2 py-1 rounded-md text-xs w-full transition duration-200" data-po="${po.poNo}" data-dept="${po.department}">
                <i class="fas fa-trash"></i> Delete
            </button>`;

            const row = document.createElement('tr');
            row.className = 'border-b border-gray-200 hover:bg-gray-50';
            row.innerHTML = `
                <td class="py-3 px-6 text-left whitespace-nowrap">${po.poNo}</td>
                <td class="py-3 px-6 text-left">${po.date}</td>
                <td class="py-3 px-6 text-left">${po.lotNo}</td>
                <td class="py-3 px-6 text-left">${po.dia}</td>
                <td class="py-3 px-6 text-left">${po.fabricKg}</td>
                <td class="py-3 px-6 text-left">${po.customer}</td>
                <td class="py-3 px-6 text-left">${po.quality}</td>
                <td class="py-3 px-6 text-left">${po.colour}</td>
                <td class="py-3 px-6 text-left capitalize ${is_admin ? '' : 'hidden'}">${po.department}</td>
                <td class="py-3 px-6 text-left font-bold text-theme-primary">${po.stage}</td>
                <td class="py-3 px-6 text-left">
                    <button class="view-history-btn text-blue-500 hover:text-blue-700 underline text-xs transition duration-200" data-po="${po.poNo}">View History</button>
                </td>
                <td class="py-3 px-6 text-left flex flex-col items-start space-y-1">${actions}</td>
            `;
            poList.appendChild(row);
        });
        if (!poList.hasChildNodes()) {
            poList.innerHTML = `<tr><td colspan="${is_admin ? 12 : 11}" class="py-4 text-center text-gray-400">No POs found for current filters.</td></tr>`;
        }
    }

    function renderPasswordRequests() {
        const listContainer = document.getElementById('passwordRequestsList');
        listContainer.innerHTML = '';
        if (pendingPasswordRequests.length === 0) {
            listContainer.innerHTML = '<p class="text-gray-500" id="noRequestsMessage">No pending requests.</p>';
            return;
        }

        pendingPasswordRequests.forEach((request, index) => {
            const requestDiv = document.createElement('div');
            requestDiv.className = 'bg-gray-50 p-4 rounded-lg shadow-sm flex flex-col md:flex-row justify-between items-start md:items-center border border-gray-200';
            requestDiv.innerHTML = `
                <div class="mb-2 md:mb-0">
                    <p class="font-semibold text-gray-800">${request.email}</p>
                    <p class="text-sm text-gray-600">Requested: ${request.date}</p>
                    <p class="text-sm text-gray-600">New Password: <span class="font-mono text-purple-700">${request.newPassword}</span></p>
                </div>
                <div class="flex space-x-2">
                    <button class="approve-password-btn bg-green-500 hover:bg-green-600 text-white px-3 py-1 rounded-md text-sm transition duration-200" data-index="${index}">Approve</button>
                    <button class="reject-password-btn bg-red-500 hover:bg-red-600 text-white px-3 py-1 rounded-md text-sm transition duration-200" data-index="${index}">Reject</button>
                </div>
            `;
            listContainer.appendChild(requestDiv);
        });
    }

    // Daily Entry Form
    document.getElementById('dailyEntryForm').addEventListener('submit', function(e) {
        e.preventDefault();
        const data = Object.fromEntries(new FormData(e.target));

        // Basic validation for fabricKg
        if (Number(data.fabricKg) <= 0) {
            showToast('Fabric KG must be a positive number.', 'error');
            return;
        }

        // Check for duplicate PO No
        if (pos.some(po => po.poNo === data.poNo)) {
            showToast(`PO Number '${data.poNo}' already exists. Please use a unique PO number.`, 'error');
            return;
        }

        data.department = currentDepartment;
        data.stage = stagesMap[currentDepartment][0];
        data.history = [{
            stage: data.stage,
            date: data.date,
            user: currentDepartment + "mgr",
            note: "PO entry created"
        }];
        pos.push(data);
        saveToLocalStorage('po_data', pos);
        e.target.reset();
        renderPOs();
        showToast('PO entry added successfully!', 'success');
    });

    // Login
    document.getElementById('loginForm').addEventListener('submit', function(e) {
        e.preventDefault();
        const dept = document.getElementById('department').value;
        const pass = document.getElementById('password').value;

        if (!dept) {
            showToast('Please select a department.', 'info');
            return;
        }
        if (LOGIN_PASSWORDS[dept] && pass === LOGIN_PASSWORDS[dept]) {
            currentDepartment = dept;
            document.getElementById('deptName').textContent = dept.charAt(0).toUpperCase() + dept.slice(1);
            document.getElementById('loginModal').style.display = 'none';
            document.getElementById('mainContent').classList.remove('hidden');

            applyDepartmentTheme(currentDepartment); // Apply theme on login

            if (currentDepartment === 'admin') {
                document.getElementById('dailyEntrySection').classList.add('hidden');
                document.getElementById('adminToolsSection').classList.remove('hidden');
                document.getElementById('departmentFilterContainer').classList.remove('hidden');
                document.getElementById('stageFilterContainer').classList.add('hidden');
                renderPasswordRequests();
            } else {
                document.getElementById('dailyEntrySection').classList.remove('hidden');
                document.getElementById('adminToolsSection').classList.add('hidden');
                document.getElementById('departmentFilterContainer').classList.add('hidden');
                document.getElementById('stageFilterContainer').classList.remove('hidden');
            }

            renderStageFilter(dept);
            renderPOs();
            showToast(`Logged in as ${dept.charAt(0).toUpperCase() + dept.slice(1)} Department.`, 'success');
        } else {
            showToast('Invalid department or password.', 'error');
        }
    });

    document.getElementById('logoutBtn').addEventListener('click', function() {
        currentDepartment = '';
        document.getElementById('mainContent').classList.add('hidden');
        document.getElementById('loginModal').style.display = 'flex';
        document.getElementById('loginForm').reset();
        document.getElementById('departmentFilter').value = '';
        document.getElementById('dailyEntrySection').classList.remove('hidden');
        document.getElementById('adminToolsSection').classList.add('hidden');
        document.getElementById('departmentFilterContainer').classList.add('hidden');
        document.getElementById('stageFilterContainer').classList.remove('hidden');
        currentSortColumn = null; // Reset sorting
        currentSortDirection = 'asc';
        applyDepartmentTheme('admin'); // Reset theme to default/admin on logout
        showToast('Logged out successfully.', 'info');
    });

    document.getElementById('applyFilters').addEventListener('click', function() {
        const stageValue = document.getElementById('stageFilter').value;
        const diaValue = document.getElementById('diaFilter').value.trim();
        const customerValue = document.getElementById('customerFilter').value.trim();
        const departmentValue = document.getElementById('departmentFilter').value;
        renderPOs(stageValue, diaValue, customerValue, departmentValue);
        showToast('Filters applied.', 'info');
    });

    // Event delegation for dynamic buttons and sorting
    document.addEventListener('click', function(e) {
        const is_admin = currentDepartment === 'admin';

        // Sorting headers
        if (e.target.classList.contains('sortable-header') || e.target.closest('.sortable-header')) {
            const header = e.target.classList.contains('sortable-header') ? e.target : e.target.closest('.sortable-header');
            const column = header.dataset.sort;

            if (currentSortColumn === column) {
                currentSortDirection = currentSortDirection === 'asc' ? 'desc' : 'asc';
            } else {
                currentSortColumn = column;
                currentSortDirection = 'asc';
            }
            sortPOs(currentSortColumn, currentSortDirection);
            return; // Prevent other click handlers from firing
        }

        // Move to next stage
        if (e.target.classList.contains('update-stage-btn') || e.target.closest('.update-stage-btn')) {
            const btn = e.target.classList.contains('update-stage-btn') ? e.target : e.target.closest('.update-stage-btn');
            const poNo = btn.dataset.po;
            const poDept = btn.dataset.dept;
            const po = pos.find(p => p.poNo === poNo);

            if (po && (is_admin || po.department === currentDepartment)) {
                const next = getNextStage(po.department, po.stage);
                if (next) {
                    const now = new Date().toISOString().slice(0,10);
                    po.stage = next;
                    po.history.push({
                        stage: next,
                        date: now,
                        user: is_admin ? 'admin' : (currentDepartment + "mgr"),
                        note: `Moved to ${next}`
                    });
                    saveToLocalStorage('po_data', pos);
                    renderPOs(
                        document.getElementById('stageFilter').value,
                        document.getElementById('diaFilter').value.trim(),
                        document.getElementById('customerFilter').value.trim(),
                        document.getElementById('departmentFilter').value
                    );
                    showToast(`PO ${poNo} moved to ${next}.`, 'success');
                } else {
                    showToast(`PO ${poNo} is already at the last stage or cannot be moved further.`, 'info');
                }
            } else {
                showToast('You do not have permission to update this PO.', 'error');
            }
        }
        // Delete PO
        if (e.target.classList.contains('delete-po-btn') || e.target.closest('.delete-po-btn')) {
            const btn = e.target.classList.contains('delete-po-btn') ? e.target : e.target.closest('.delete-po-btn');
            const poNoToDelete = btn.dataset.po;
            const poDept = btn.dataset.dept;

            if (is_admin || poDept === currentDepartment) {
                if (confirm(`Are you sure you want to delete PO ${poNoToDelete}? This action cannot be undone.`)) {
                    const initialLength = pos.length;
                    pos = pos.filter(po => !(po.poNo === poNoToDelete));
                    saveToLocalStorage('po_data', pos);
                    if (pos.length < initialLength) {
                        renderPOs(
                            document.getElementById('stageFilter').value,
                            document.getElementById('diaFilter').value.trim(),
                            document.getElementById('customerFilter').value.trim(),
                            document.getElementById('departmentFilter').value
                        );
                        showToast(`PO ${poNoToDelete} deleted successfully.`, 'success');
                    } else {
                        showToast(`Could not find PO ${poNoToDelete} to delete.`, 'error');
                    }
                }
            } else {
                showToast('You do not have permission to delete this PO.', 'error');
            }
        }
        // Open Change PO Stage Modal
        if (e.target.classList.contains('change-stage-btn') || e.target.closest('.change-stage-btn')) {
            const btn = e.target.classList.contains('change-stage-btn') ? e.target : e.target.closest('.change-stage-btn');
            const poNoToChange = btn.dataset.po;
            const poDept = btn.dataset.dept;
            const po = pos.find(p => p.poNo === poNoToChange);

            if (po && (is_admin || po.department === currentDepartment)) {
                document.getElementById('modalPoNo').textContent = po.poNo;
                document.getElementById('hiddenPoNo').value = po.poNo;
                document.getElementById('hiddenPoDept').value = po.department;
                const newPoStageSelect = document.getElementById('newPoStage');
                newPoStageSelect.innerHTML = '';

                const stagesForPoDept = stagesMap[po.department];
                if (stagesForPoDept) {
                    stagesForPoDept.forEach(stage => {
                        const option = document.createElement('option');
                        option.value = stage;
                        option.textContent = stage;
                        if (stage === po.stage) {
                            option.selected = true;
                        }
                        newPoStageSelect.appendChild(option);
                    });
                } else {
                    newPoStageSelect.innerHTML = '<option value="">No stages found for this department</option>';
                }

                document.getElementById('stageNote').value = '';
                document.getElementById('changePoStageModal').classList.remove('hidden');
            } else {
                showToast('You do not have permission to change the stage of this PO.', 'error');
            }
        }

        // Open PO History Modal
        if (e.target.classList.contains('view-history-btn') || e.target.closest('.view-history-btn')) {
            const btn = e.target.classList.contains('view-history-btn') ? e.target : e.target.closest('.view-history-btn');
            const poNo = btn.dataset.po;
            const po = pos.find(p => p.poNo === poNo);

            if (po) {
                document.getElementById('historyModalPoNo').textContent = po.poNo;
                const historyContent = document.getElementById('historyContent');
                historyContent.innerHTML = '';

                if (po.history && po.history.length > 0) {
                    po.history.forEach(entry => {
                        const historyEntryDiv = document.createElement('div');
                        historyEntryDiv.className = 'bg-gray-50 p-3 rounded-lg shadow-sm border border-gray-200';
                        historyEntryDiv.innerHTML = `
                            <p class="font-bold text-lg text-theme-primary">${entry.stage}</p>
                            <p class="text-sm text-gray-600">Date: ${entry.date}</p>
                            <p class="text-sm text-gray-600">User: ${entry.user}</p>
                            <p class="text-sm text-gray-800">Note: ${entry.note || 'N/A'}</p>
                        `;
                        historyContent.appendChild(historyEntryDiv);
                    });
                } else {
                    historyContent.innerHTML = '<p class="text-gray-500">No history available for this PO.</p>';
                }
                document.getElementById('poHistoryModal').classList.remove('hidden');
            }
        }

        // Admin: Approve Password Request
        if (e.target.classList.contains('approve-password-btn') || e.target.closest('.approve-password-btn')) {
            const btn = e.target.classList.contains('approve-password-btn') ? e.target : e.target.closest('.approve-password-btn');
            const index = parseInt(btn.dataset.index);
            if (index >= 0 && index < pendingPasswordRequests.length) {
                const request = pendingPasswordRequests[index];
                // In a real system, you'd update the user's password here.
                LOGIN_PASSWORDS[request.department] = request.newPassword; // Update the in-memory password
                saveToLocalStorage('login_passwords', LOGIN_PASSWORDS); // Save updated passwords
                showToast(`Password for ${request.email} approved. New password: ${request.newPassword}. (In a real system, this would be sent to the user and securely stored.)`, 'success', 5000);
                pendingPasswordRequests.splice(index, 1);
                saveToLocalStorage('pending_password_requests', pendingPasswordRequests);
                renderPasswordRequests();
            }
        }

        // Admin: Reject Password Request
        if (e.target.classList.contains('reject-password-btn') || e.target.closest('.reject-password-btn')) {
            const btn = e.target.classList.contains('reject-password-btn') ? e.target : e.target.closest('.reject-password-btn');
            const index = parseInt(btn.dataset.index);
            if (index >= 0 && index < pendingPasswordRequests.length) {
                const request = pendingPasswordRequests[index];
                showToast(`Password change request for ${request.email} rejected.`, 'info');
                pendingPasswordRequests.splice(index, 1);
                saveToLocalStorage('pending_password_requests', pendingPasswordRequests);
                renderPasswordRequests();
            }
        }
    });

    // Change Password Button (Login Modal)
    document.getElementById('changePasswordBtn').addEventListener('click', function() {
        document.getElementById('changePasswordModal').classList.remove('hidden');
    });

    // Cancel Change Password
    document.getElementById('cancelChangePassword').addEventListener('click', function() {
        document.getElementById('changePasswordModal').classList.add('hidden');
        document.getElementById('changePasswordForm').reset();
    });

    // Change Password Submit (Request)
    document.getElementById('changePasswordForm').addEventListener('submit', function(e) {
        e.preventDefault();
        const email = document.getElementById('deptEmail').value.trim();
        const newPass = document.getElementById('newPassword').value;
        if (!email || !newPass) {
            showToast('Please enter email and new password.', 'error');
            return;
        }

        // Attempt to derive department from email (simple heuristic)
        let departmentFromEmail = '';
        const emailParts = email.split('@');
        if (emailParts.length > 0) {
            const deptName = emailParts[0].toLowerCase().replace('.', ''); // e.g., your.department -> yourdepartment
            for (const deptKey in stagesMap) {
                if (deptName.includes(deptKey)) {
                    departmentFromEmail = deptKey;
                    break;
                }
            }
            if (!departmentFromEmail && deptName.includes('admin')) { // Special case for admin
                departmentFromEmail = 'admin';
            }
        }

        if (!departmentFromEmail) {
            showToast('Could not determine department from email. Please ensure email contains department name (e.g., spinning@example.com).', 'error', 5000);
            return;
        }

        pendingPasswordRequests.push({
            email: email,
            newPassword: newPass,
            date: new Date().toISOString().slice(0,10),
            department: departmentFromEmail // Store the derived department
        });
        saveToLocalStorage('pending_password_requests', pendingPasswordRequests);

        showToast(`Password change request sent for ${email}. Admin will review and send new password to your email after approval.`, 'info', 5000);
        document.getElementById('changePasswordModal').classList.add('hidden');
        document.getElementById('changePasswordForm').reset();
    });

    // Report Modal
    document.getElementById('reportBtn').addEventListener('click', function() {
        document.getElementById('reportModal').classList.remove('hidden');
        renderReport();
    });
    document.getElementById('closeReport').addEventListener('click', function() {
        document.getElementById('reportModal').classList.add('hidden');
    });

    // Close PO History Modal
    document.getElementById('closeHistoryModal').addEventListener('click', function() {
        document.getElementById('poHistoryModal').classList.add('hidden');
    });

    // Change PO Stage Form Submit
    document.getElementById('changePoStageForm').addEventListener('submit', function(e) {
        e.preventDefault();
        const poNo = document.getElementById('hiddenPoNo').value;
        const poDept = document.getElementById('hiddenPoDept').value;
        const newStage = document.getElementById('newPoStage').value;
        const stageNote = document.getElementById('stageNote').value.trim();

        const po = pos.find(p => p.poNo === poNo);

        if (po) {
            const now = new Date().toISOString().slice(0,10);
            po.stage = newStage;
            po.history.push({
                stage: newStage,
                date: now,
                user: currentDepartment === 'admin' ? 'admin' : (currentDepartment + "mgr"),
                note: stageNote || `Stage manually changed to ${newStage}`
            });
            saveToLocalStorage('po_data', pos);
            renderPOs(
                document.getElementById('stageFilter').value,
                document.getElementById('diaFilter').value.trim(),
                document.getElementById('customerFilter').value.trim(),
                document.getElementById('departmentFilter').value
            );
            showToast(`PO ${poNo} stage updated to ${newStage}.`, 'success');
            document.getElementById('changePoStageModal').classList.add('hidden');
        } else {
            showToast('Error: PO not found for stage change.', 'error');
        }
    });

    // Cancel Change PO Stage
    document.getElementById('cancelChangePoStage').addEventListener('click', function() {
        document.getElementById('changePoStageModal').classList.add('hidden');
    });


    function renderReport() {
        const reportPOs = currentDepartment === 'admin' ? pos : pos.filter(po => po.department === currentDepartment);

        const totalReceived = reportPOs.length;
        const totalProduction = reportPOs.reduce((acc, po) => acc + Number(po.fabricKg), 0);
        const totalCompleted = reportPOs.filter(po => po.stage === "Completed").length;
        const poBalance = reportPOs.filter(po => po.stage !== "Completed").length;
        let diaBalance = {};
        reportPOs.filter(po => po.stage !== "Completed").forEach(po => {
            diaBalance[po.dia] = (diaBalance[po.dia] || 0) + 1;
        });
        let diaText = Object.entries(diaBalance).map(([dia, cnt]) => `${dia}: ${cnt} PO(s)`).join(", ");
        let daysRemain = poBalance;

        document.getElementById('totalReceived').textContent = totalReceived;
        document.getElementById('totalProduction').textContent = totalProduction + " KG";
        document.getElementById('totalCompleted').textContent = totalCompleted;
        document.getElementById('poBalance').textContent = diaText || "None";
        document.getElementById('daysRemain').textContent = daysRemain + " day(s)";

        let stageCounts = {};
        if (currentDepartment === 'admin') {
            const allStages = new Set();
            Object.values(stagesMap).forEach(deptStages => {
                deptStages.forEach(stage => allStages.add(stage));
            });
            Array.from(allStages).sort().forEach(s => stageCounts[s] = 0);
        } else {
            stagesMap[currentDepartment].forEach(s => stageCounts[s] = 0);
        }

        reportPOs.forEach(po => stageCounts[po.stage] = (stageCounts[po.stage] || 0) + 1);

        const ctx = document.getElementById('poGraph').getContext('2d');
        if(window.poChart) window.poChart.destroy();
        window.poChart = new Chart(ctx, {
            type: 'bar',
            data: {
                labels: Object.keys(stageCounts),
                datasets: [{
                    label: 'POs by Stage',
                    data: Object.values(stageCounts),
                    backgroundColor: 'var(--primary-color)', /* Use primary theme color */
                    borderColor: 'var(--primary-dark-color)', /* Use primary dark theme color */
                    borderWidth: 1
                }]
            },
            options: {
                scales: { y: { beginAtZero: true } },
                plugins: { legend: { display: false } }
            }
        });
    }

    // --- Excel Export Functionality ---
    function exportToCsv(filename, rows) {
        const csvFile = rows.map(e => e.join(",")).join("\n");
        const blob = new Blob([csvFile], { type: 'text/csv;charset=utf-8;' });
        if (navigator.msSaveBlob) { // IE 10+
            navigator.msSaveBlob(blob, filename);
        } else {
            const link = document.createElement("a");
            if (link.download !== undefined) { // Feature detection
                // Browsers that support HTML5 download attribute
                const url = URL.createObjectURL(blob);
                link.setAttribute("href", url);
                link.setAttribute("download", filename);
                link.style.visibility = 'hidden';
                document.body.appendChild(link);
                link.click();
                document.body.removeChild(link);
            }
        }
        showToast(`Exported ${filename} successfully!`, 'success');
    }

    document.getElementById('downloadExcelBtn').addEventListener('click', function() {
        let dataToExport = [];
        let filename = '';

        if (currentDepartment === 'admin') {
            // Admin exports all data
            filename = 'All_Departments_PO_Data.csv';
            // Define headers for admin export, including 'Department'
            const headers = ["PO No", "Date", "Lot No", "DIA", "Fabric KG", "Customer", "Quality", "Colour", "Department", "Current Stage", "History"];
            dataToExport.push(headers);

            pos.forEach(po => {
                const historyString = po.history.map(h => `${h.date} - ${h.stage} (${h.user}): ${h.note || 'N/A'}`).join(" | ");
                dataToExport.push([
                    po.poNo, po.date, po.lotNo, po.dia, po.fabricKg,
                    po.customer, po.quality, po.colour, po.department, po.stage, historyString
                ]);
            });
        } else {
            // Department exports their own data
            filename = `${currentDepartment.charAt(0).toUpperCase() + currentDepartment.slice(1)}_PO_Data.csv`;
            // Define headers for department export (no 'Department' column)
            const headers = ["PO No", "Date", "Lot No", "DIA", "Fabric KG", "Customer", "Quality", "Colour", "Current Stage", "History"];
            dataToExport.push(headers);

            pos.filter(po => po.department === currentDepartment).forEach(po => {
                const historyString = po.history.map(h => `${h.date} - ${h.stage} (${h.user}): ${h.note || 'N/A'}`).join(" | ");
                dataToExport.push([
                    po.poNo, po.date, po.lotNo, po.dia, po.fabricKg,
                    po.customer, po.quality, po.colour, po.stage, historyString
                ]);
            });
        }

        if (dataToExport.length > 1) { // Check if there's data beyond just headers
            exportToCsv(filename, dataToExport);
        } else {
            showToast('No data available to export for this view.', 'info');
        }
    });

    // Initial theme application (for login modal)
    applyDepartmentTheme('admin'); // Default theme for login screen

    // Show login by default
    document.getElementById('mainContent').classList.add('hidden');
    document.getElementById('loginModal').style.display = 'flex';
    </script>
</body>
</html>

