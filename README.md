# Bariwala<!DOCTYPE html>
<html lang="bn">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>বাসা ভাড়া হিসাব রক্ষক</title>

    <style>
        /* ======================== CSS কোড ======================== */
        body { 
            font-family: Arial, sans-serif; 
            margin: 0; 
            padding: 10px; 
            background-color: #f4f4f4;
        }
        h1 { text-align: center; color: #333; }
        
        /* বাটন ও কন্ট্রোল সেকশন */
        .controls, .month-buttons { margin-bottom: 15px; padding: 10px; background-color: #fff; border-radius: 8px; box-shadow: 0 2px 4px rgba(0,0,0,0.1); }
        .month-buttons button { 
            padding: 10px 15px; 
            margin: 5px 2px; 
            cursor: pointer; 
            border: 1px solid #ccc;
            background-color: #f9f9f9;
            border-radius: 5px;
            transition: background-color 0.2s;
        }
        .month-buttons button:hover { background-color: #ddd; }
        .month-buttons .active { background-color: #007bff; color: white; border-color: #007bff; }
        
        /* ডেটা সেকশন ও টেবিল */
        .rent-data-section { padding: 10px; background-color: #fff; border-radius: 8px; box-shadow: 0 2px 4px rgba(0,0,0,0.1); overflow-x: auto; }
        table { 
            width: 100%; 
            border-collapse: collapse; 
            min-width: 800px; /* ছোট স্ক্রিনে যাতে কলামগুলো চাপা না খায় */
        }
        th, td { 
            border: 1px solid #ddd; 
            padding: 5px; 
            text-align: center; 
            white-space: nowrap;
        }
        th { background-color: #e9e9e9; }

        /* ইনপুট ফিল্ডের স্টাইল */
        input[type="text"], input[type="number"], input[type="date"] {
            border: none;
            padding: 5px;
            width: 100%;
            box-sizing: border-box;
            background: transparent;
            text-align: center;
        }
        
        /* কলামের চওড়া নিয়ন্ত্রণ */
        .col-name input { width: 98%; text-align: left; } /* নাম ঘর চওড়া */
        .col-mobile input { width: 98%; } /* লেখার সাথে সাথে কলামের আকার ছোট বড় হবে - এটি ব্রাউজার/webview অটোমেটিক করবে */
        .col-rent, .col-deposit, .col-due { width: 100px; } 

        /* মোট হিসাব */
        tfoot input { font-weight: bold; background-color: #e9f5ff; }
        tfoot td { text-align: center; font-weight: bold; }
        
        .save-button { 
            margin-top: 20px; 
            padding: 12px 25px; 
            font-size: 16px; 
            background-color: #28a745; 
            color: white; 
            border: none; 
            border-radius: 5px; 
            cursor: pointer;
            display: block;
            width: 100%;
        }
        .save-button:hover { background-color: #218838; }
    </style>
</head>
<body>

    <h1>বাসা ভাড়া হিসাব রক্ষক</h1>

    <div class="controls">
        <label for="current-year">বছর:</label>
        <input type="number" id="current-year" value="2025" min="2024" onchange="changeYear(this.value)">
        <button onclick="addYear()">বছর যোগ করুন</button>
        <button onclick="removeYear()">বছর রিমুভ করুন</button>
        <hr>
        <button onclick="addRoom()">রুম যোগ করুন</button>
        <button onclick="removeLastRoom()">শেষের রুমটি রিমুভ করুন</button>
    </div>

    <div class="month-buttons" id="month-buttons">
        </div>
    
    <div class="rent-data-section">
        <h2 id="month-heading" style="text-align: center;">জানুয়ারি মাসের ভাড়া</h2>
        
        <table>
            <thead>
                <tr>
                    <th>Room No<br>(রুম নং)</th>
                    <th class="col-name">Name<br>(নাম)</th>
                    <th>Mobile<br>(মোবাইল)</th>
                    <th class="col-rent">Rent<br>(ভাড়া)</th>
                    <th>Date<br>(তারিখ)</th>
                    <th class="col-deposit">Deposit<br>(আদায়)</th>
                    <th class="col-due">Due<br>(বকেয়া)</th>
                </tr>
            </thead>
            <tbody id="rent-table-body">
                </tbody>
            <tfoot>
                <tr>
                    <td colspan="3" style="text-align: right;">মোট:</td>
                    <td><input type="number" id="total-rent-due" readonly></td>
                    <td></td>
                    <td><input type="number" id="total-deposit" readonly></td>
                    <td><input type="number" id="total-due" readonly></td>
                </tr>
            </tfoot>
        </table>

        <button class="save-button" onclick="saveData()">সব তথ্য সেভ করুন</button>
    </div>

    <script>
        const MONTHS = ["জানুয়ারি", "ফেব্রুয়ারি", "মার্চ", "এপ্রিল", "মে", "জুন", "জুলাই", "আগস্ট", "সেপ্টেম্বর", "অক্টোবর", "নভেম্বর", "ডিসেম্বর"];
        const DEFAULT_ROOMS = 14;
        let currentYear = new Date().getFullYear();
        let currentMonthIndex = 0; 
        let rentalData = {}; // সমস্ত ডেটা এই অবজেক্টে থাকবে

        // **১. ডেটা লোড ও প্রাথমিক সেটিং**
        document.addEventListener('DOMContentLoaded', () => {
            document.getElementById('current-year').value = currentYear;
            loadDataFromLocalStore();
            createMonthButtons();
            loadMonthData(currentYear, currentMonthIndex);
        });

        // **২. লোকাল স্টোরেজ থেকে ডেটা লোড**
        function loadDataFromLocalStore() {
            const data = localStorage.getItem('rentalData');
            if (data) {
                rentalData = JSON.parse(data);
            } else {
                initializeDefaultData(currentYear);
            }
        }

        // **৩. ডিফল্ট ডেটা তৈরি (১৪ রুম)**
        function initializeDefaultData(year) {
            // যদি বছর না থাকে, তবে তৈরি করুন
            if (!rentalData[year]) {
                rentalData[year] = [];
            }
            
            // ১২ মাসের জন্য ডেটা ইনিশিয়ালাইজ করা
            for (let m = 0; m < 12; m++) {
                if (!rentalData[year][m] || rentalData[year][m].length === 0) {
                    let monthData = [];
                    for (let r = 1; r <= DEFAULT_ROOMS; r++) {
                        monthData.push({
                            room: r, name: '', mobile: '', rent: '', date: '', deposit: '', due: ''
                        });
                    }
                    rentalData[year][m] = monthData;
                }
            }
        }

        // **৪. মাস বাটন তৈরি ও নির্বাচন**
        function createMonthButtons() {
            const container = document.getElementById('month-buttons');
            container.innerHTML = '';
            MONTHS.forEach((month, index) => {
                const button = document.createElement('button');
                button.textContent = month;
                button.className = index === currentMonthIndex ? 'active' : '';
                button.onclick = () => selectMonth(index, button);
                container.appendChild(button);
            });
        }

        function selectMonth(index, button) {
            currentMonthIndex = index;
            document.getElementById('month-heading').textContent = `${MONTHS[index]} মাসের ভাড়া`;
            
            // Active বাটন পরিবর্তন করা
            document.querySelectorAll('.month-buttons button').forEach(btn => btn.classList.remove('active'));
            button.classList.add('active');
            
            loadMonthData(currentYear, currentMonthIndex);
        }
        
        function changeYear(year) {
            const yearInt = parseInt(year);
            if (rentalData[yearInt]) {
                currentYear = yearInt;
            } else {
                // নতুন বছর যোগ করার জন্য বার্তা দিন
                alert(`${yearInt} সালের কোনো ডেটা পাওয়া যায়নি। যোগ করতে 'বছর যোগ করুন' বাটন ব্যবহার করুন।`);
                // আগের বছরে ফিরে যাওয়া
                document.getElementById('current-year').value = currentYear;
            }
            loadMonthData(currentYear, currentMonthIndex);
        }

        // **৫. টেবিল রো তৈরি করা**
        function loadMonthData(year, monthIndex) {
            const tableBody = document.getElementById('rent-table-body');
            tableBody.innerHTML = '';
            
            // নিশ্চিত করুন যে ডেটা স্ট্রাকচার ঠিক আছে
            if (!rentalData[year]) initializeDefaultData(year);
            let monthData = rentalData[year][monthIndex];
            
            // যদি মাসের ডেটা না থাকে, ডিফল্ট রুম ডেটা নিন
            if (!monthData || monthData.length === 0) {
                 const currentRoomCount = rentalData[year][0].length; // প্রথম মাসের রুম সংখ্যা
                 monthData = [];
                 for (let r = 1; r <= currentRoomCount; r++) {
                    monthData.push({ room: r, name: '', mobile: '', rent: '', date: '', deposit: '', due: '' });
                 }
                 rentalData[year][monthIndex] = monthData;
            }

            monthData.forEach((room, index) => {
                const row = tableBody.insertRow();
                
                // রুম নং
                row.insertCell().innerHTML = `<input type="text" value="${room.room}" oninput="updateRoomData(${index}, 'room', this.value)">`; 
                // নাম 
                row.insertCell().className = 'col-name';
                row.insertCell().innerHTML = `<input type="text" value="${room.name}" oninput="updateRoomData(${index}, 'name', this.value)">`;
                // মোবাইল
                row.insertCell().innerHTML = `<input type="text" value="${room.mobile}" oninput="updateRoomData(${index}, 'mobile', this.value)">`;
                // ভাড়া
                row.insertCell().className = 'col-rent';
                row.insertCell().innerHTML = `<input type="number" value="${room.rent}" oninput="updateRoomData(${index}, 'rent', this.value); calculateTotals()">`;
                // তারিখ
                row.insertCell().innerHTML = `<input type="date" value="${room.date}" oninput="updateRoomData(${index}, 'date', this.value)">`;
                // আদায়
                row.insertCell().className = 'col-deposit';
                row.insertCell().innerHTML = `<input type="number" value="${room.deposit}" oninput="updateRoomData(${index}, 'deposit', this.value); calculateTotals()">`;
                // বকেয়া (স্বয়ংক্রিয়ভাবে ক্যালকুলেট হবে)
                row.insertCell().className = 'col-due';
                row.insertCell().innerHTML = `<input type="number" value="${room.due}" readonly>`;
            });
            calculateTotals(); 
        }

        // **৬. ডেটা আপডেট ও ক্যালকুলেশন**
        function updateRoomData(rowIndex, columnKey, value) {
            const yearData = rentalData[currentYear];
            if (!yearData || !yearData[currentMonthIndex]) return;

            let finalValue = value;
            if (['rent', 'deposit'].includes(columnKey)) {
                finalValue = parseFloat(value) || ''; 
            }

            yearData[currentMonthIndex][rowIndex][columnKey] = finalValue;
            
            // বকেয়া হিসাব
            const roomData = yearData[currentMonthIndex][rowIndex];
            const rent = parseFloat(roomData.rent) || 0;
            const deposit = parseFloat(roomData.deposit) || 0;
            roomData.due = (rent - deposit).toFixed(0); 
            
            // বকেয়ার ঘরটি আপডেট করা
            const tableBody = document.getElementById('rent-table-body');
            const row = tableBody.rows[rowIndex];
            if (row && row.cells.length >= 7) {
                row.cells[6].querySelector('input').value = roomData.due;
            }

            calculateTotals();
        }

        // মোট হিসাব করা
        function calculateTotals() {
            const monthData = rentalData[currentYear] ? rentalData[currentYear][currentMonthIndex] : [];
            let totalRent = 0;
            let totalDeposit = 0;
            let totalDue = 0;

            monthData.forEach(room => {
                const rent = parseFloat(room.rent) || 0;
                const deposit = parseFloat(room.deposit) || 0;
                totalRent += rent;
                totalDeposit += deposit;
                totalDue += (rent - deposit);
            });

            document.getElementById('total-rent-due').value = totalRent.toFixed(0);
            document.getElementById('total-deposit').value = totalDeposit.toFixed(0);
            document.getElementById('total-due').value = totalDue.toFixed(0);
        }
        
        // **৭. রুম এড/রিমুভ**
        function addRoom() {
            // অন্য মাসগুলোর রুম সংখ্যার উপর ভিত্তি করে নতুন রুম নং বের করা
            const firstMonthData = rentalData[currentYear][0];
            const nextRoomNo = firstMonthData ? firstMonthData.length + 1 : 1; 

            // এই বছরের সব মাসের ডেটা আপডেট করা
            for (let m = 0; m < 12; m++) {
                if(rentalData[currentYear][m]) {
                    rentalData[currentYear][m].push({
                        room: nextRoomNo, name: '', mobile: '', rent: '', date: '', deposit: '', due: ''
                    });
                } else {
                     // মাসের ডেটা না থাকলে, সেই মাসকেও ইনিশিয়ালাইজ করা
                     rentalData[currentYear][m] = [{
                         room: nextRoomNo, name: '', mobile: '', rent: '', date: '', deposit: '', due: ''
                     }];
                }
            }
            loadMonthData(currentYear, currentMonthIndex);
            saveData(); // রুম কাঠামো পরিবর্তন হলে সেভ করা ভালো
            alert(`${nextRoomNo} নং রুমটি সব মাসে যোগ করা হয়েছে।`);
        }

        function removeLastRoom() {
            const firstMonthData = rentalData[currentYear][0];
            if (!firstMonthData || firstMonthData.length <= 1) {
                alert("কমপক্ষে একটি রুম থাকতে হবে!");
                return;
            }

            const roomNoToRemove = firstMonthData.length;
            const confirmRemove = confirm(`${roomNoToRemove} নং রুমটির ডেটা সব মাস থেকে রিমুভ করতে চান?`);
            
            if (confirmRemove) {
                // অন্য মাস থেকেও শেষ রুমটি রিমুভ হবে
                for (let m = 0; m < 12; m++) {
                    if(rentalData[currentYear][m] && rentalData[currentYear][m].length > 0) {
                        rentalData[currentYear][m].pop(); 
                    }
                }
                loadMonthData(currentYear, currentMonthIndex);
                saveData();
                alert(`${roomNoToRemove} নং রুমটি রিমুভ করা হয়েছে।`);
            }
        }

        // **৮. বছর এড/রিমুভ**
        function addYear() {
            const newYear = prompt("নতুন বছর যোগ করুন (যেমন: 2026):");
            const yearInt = parseInt(newYear);
            if (yearInt && !rentalData[yearInt]) {
                initializeDefaultData(yearInt);
                currentYear = yearInt;
                document.getElementById('current-year').value = yearInt;
                loadMonthData(currentYear, currentMonthIndex);
                saveData();
                alert(`${yearInt} বছর যোগ করা হয়েছে এবং ডিফল্ট ${rentalData[yearInt][0].length}টি রুম সেট করা হয়েছে।`);
            } else if (yearInt) {
                alert(`${yearInt} বছরটি ইতিমধ্যেই আছে বা ইনপুট সঠিক নয়।`);
            }
        }

        function removeYear() {
            const yearToRemove = document.getElementById('current-year').value;
            if (Object.keys(rentalData).length <= 1) {
                alert("কমপক্ষে একটি বছর থাকতে হবে!");
                return;
            }
            if (rentalData[yearToRemove] && confirm(`আপনি কি ${yearToRemove} বছরটির সব ডেটা রিমুভ করতে চান?`)) {
                delete rentalData[yearToRemove];
                
                // অন্য বছরকে বর্তমান বছর হিসেবে সেট করা
                const remainingYears = Object.keys(rentalData).map(y => parseInt(y)).sort();
                if (remainingYears.length > 0) {
                    currentYear = remainingYears[0];
                    document.getElementById('current-year').value = currentYear;
                }
                loadMonthData(currentYear, currentMonthIndex);
                saveData();
                alert(`${yearToRemove} বছরটির ডেটা রিমুভ করা হয়েছে।`);
            } else {
                 // Nothing
            }
        }


        // **৯. সেভ ডেটা**
        function saveData() {
            // সমস্ত ডেটা লোকাল স্টোরেজে সেভ করা
            localStorage.setItem('rentalData', JSON.stringify(rentalData));
            alert('ডেটা সফলভাবে সেভ করা হয়েছে!');
        }

    </script>
</body>
</html>
