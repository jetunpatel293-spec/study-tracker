# study-tracker<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>My Study Tracker</title>
    <style>
        body { font-family: Arial, sans-serif; margin: 0; padding: 0; background-color: #f4f4f4; color: #333; }
        header { background-color: #4CAF50; color: white; text-align: center; padding: 1rem; }
        nav { background-color: #333; padding: 0.5rem; text-align: center; }
        nav a { color: white; margin: 0 1rem; text-decoration: none; padding: 0.5rem; border-radius: 5px; }
        nav a:hover { background-color: #555; }
        .container { max-width: 800px; margin: 0 auto; padding: 1rem; }
        section { margin-bottom: 2rem; padding: 1rem; background-color: white; border-radius: 8px; box-shadow: 0 2px 5px rgba(0,0,0,0.1); }
        h2 { color: #4CAF50; }
        input, button, textarea { padding: 0.5rem; margin: 0.5rem 0; border: 1px solid #ccc; border-radius: 4px; }
        button { background-color: #4CAF50; color: white; cursor: pointer; }
        button:hover { background-color: #45a049; }
        ul { list-style-type: none; padding: 0; }
        li { padding: 0.5rem; border-bottom: 1px solid #eee; display: flex; justify-content: space-between; align-items: center; }
        .completed { text-decoration: line-through; color: #888; }
        .chart-container { width: 100%; height: 300px; }
        @media (max-width: 600px) { nav a { display: block; margin: 0.5rem 0; } .container { padding: 0.5rem; } }
    </style>
</head>
<body>
    <header>
        <h1>My Study Tracker</h1>
        <p>Welcome, eduflow! Track your progress and stay motivated.</p>
    </header>
    <nav>
        <a href="#home">Home</a>
        <a href="#todo">To-Do List</a>
        <a href="#tracker">Study Tracker</a>
        <a href="#goals">Goals</a>
        <a href="#progress">Progress</a>
    </nav>
    <div class="container">
        <section id="home">
            <h2>Home</h2>
            <p>Today's Goals:</p>
            <ul id="todays-goals"></ul>
            <p id="quote">Motivational Quote: "The only way to do great work is to love what you do." - Steve Jobs</p>
        </section>
        <section id="todo">
            <h2>To-Do List</h2>
            <input type="text" id="task-input" placeholder="Add a new task">
            <button onclick="addTask()">Add Task</button>
            <ul id="task-list"></ul>
        </section>
        <section id="tracker">
            <h2>Study Tracker</h2>
            <input type="text" id="subject-input" placeholder="Subject (e.g., Math)">
            <input type="number" id="hours-input" placeholder="Hours studied" min="0" step="0.5">
            <button onclick="logStudy()">Log Study</button>
            <h3>Today's Log</h3>
            <ul id="daily-log"></ul>
            <h3>Weekly Summary</h3>
            <div id="weekly-summary"></div>
        </section>
        <section id="goals">
            <h2>Goals</h2>
            <h3>Short-Term Goals</h3>
            <textarea id="short-goals" placeholder="E.g., Finish math homework this week"></textarea>
            <button onclick="saveGoals('short')">Save Short-Term Goals</button>
            <p id="short-display"></p>
            <h3>Long-Term Goals</h3>
            <textarea id="long-goals" placeholder="E.g., Score 90% in 10th board exams"></textarea>
            <button onclick="saveGoals('long')">Save Long-Term Goals</button>
            <p id="long-display"></p>
        </section>
        <section id="progress">
            <h2>Progress</h2>
            <h3>Study Hours by Subject (This Week)</h3>
            <div class="chart-container"><canvas id="subject-chart"></canvas></div>
            <h3>Weekly Study Hours</h3>
            <div class="chart-container"><canvas id="weekly-chart"></canvas></div>
        </section>
    </div>
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
    <script>
        // Motivational quotes array
        const quotes = [
            "The only way to do great work is to love what you do. - Steve Jobs",
            "Believe you can and you're halfway there. - Theodore Roosevelt",
            "The future belongs to those who believe in the beauty of their dreams. - Eleanor Roosevelt",
            "Study is not just about passing exams; it's about building a better future. - Anonymous"
        ];

        // Load data from localStorage
        let tasks = JSON.parse(localStorage.getItem('tasks')) || [];
        let studyLogs = JSON.parse(localStorage.getItem('studyLogs')) || [];
        let shortGoals = localStorage.getItem('shortGoals') || '';
        let longGoals = localStorage.getItem('longGoals') || '';

        // Initialize
        document.addEventListener('DOMContentLoaded', () => {
            renderTasks();
            renderDailyLog();
            renderWeeklySummary();
            renderGoals();
            renderTodaysGoals();
            updateQuote();
            renderCharts();
        });

        // Home: Update quote
        function updateQuote() {
            document.getElementById('quote').textContent = 'Motivational Quote: ' + quotes[Math.floor(Math.random() * quotes.length)];
        }

        // To-Do List
        function addTask() {
            const input = document.getElementById('task-input');
            if (input.value.trim()) {
                tasks.push({ text: input.value, completed: false, today: false });
                localStorage.setItem('tasks', JSON.stringify(tasks));
                renderTasks();
                input.value = '';
            }
        }

        function renderTasks() {
            const list = document.getElementById('task-list');
            list.innerHTML = '';
            tasks.forEach((task, index) => {
                const li = document.createElement('li');
                li.innerHTML = `
                    <span class="${task.completed ? 'completed' : ''}">${task.text}</span>
                    <div>
                        <input type="checkbox" ${task.completed ? 'checked' : ''} onchange="toggleComplete(${index})">
                        <input type="checkbox" ${task.today ? 'checked' : ''} onchange="toggleToday(${index})"> Today
                        <button onclick="editTask(${index})">Edit</button>
                        <button onclick="deleteTask(${index})">Delete</button>
                    </div>
                `;
                list.appendChild(li);
            });
            renderTodaysGoals();
        }

        function toggleComplete(index) {
            tasks[index].completed = !tasks[index].completed;
            localStorage.setItem('tasks', JSON.stringify(tasks));
            renderTasks();
        }

        function toggleToday(index) {
            tasks[index].today = !tasks[index].today;
            localStorage.setItem('tasks', JSON.stringify(tasks));
            renderTasks();
        }

        function editTask(index) {
            const newText = prompt('Edit task:', tasks[index].text);
            if (newText) {
                tasks[index].text = newText;
                localStorage.setItem('tasks', JSON.stringify(tasks));
                renderTasks();
            }
        }

        function deleteTask(index) {
            tasks.splice(index, 1);
            localStorage.setItem('tasks', JSON.stringify(tasks));
            renderTasks();
        }

        function renderTodaysGoals() {
            const list = document.getElementById('todays-goals');
            list.innerHTML = '';
            tasks.filter(task => task.today && !task.completed).forEach(task => {
                const li = document.createElement('li');
                li.textContent = task.text;
                list.appendChild(li);
            });
        }

        // Study Tracker
        function logStudy() {
            const subject = document.getElementById('subject-input').value.trim();
            const hours = parseFloat(document.getElementById('hours-input').value);
            if (subject && hours > 0) {
                const today = new Date().toDateString();
                studyLogs.push({ subject, hours, date: today });
                localStorage.setItem('studyLogs', JSON.stringify(studyLogs));
                renderDailyLog();
                renderWeeklySummary();
                renderCharts();
                document.getElementById('subject-input').value = '';
                document.getElementById('hours-input').value = '';
            }
        }

        function renderDailyLog() {
            const list = document.getElementById('daily-log');
            list.innerHTML = '';
            const today = new Date().toDateString();
            studyLogs.filter(log => log.date === today).forEach(log => {
                const li = document.createElement('li');
                li.textContent = `${log.subject}: ${log.hours} hours`;
                list.appendChild(li);
            });
        }

        function renderWeeklySummary() {
            const summary = document.getElementById('weekly-summary');
            const weekAgo = new Date();
            weekAgo.setDate(weekAgo.getDate() - 7);
            const weeklyLogs = studyLogs.filter(log => new Date(log.date) >= weekAgo);
            const totalHours = weeklyLogs.reduce((sum, log) => sum + log.hours, 0);
            summary.textContent = `Total hours studied this week: ${totalHours}`;
        }

        // Goals
        function saveGoals(type) {
            if (type === 'short') {
                shortGoals = document.getElementById('short-goals').value;
                localStorage.setItem('shortGoals', shortGoals);
            } else {
                longGoals = document.getElementById('long-goals').value;
                localStorage.setItem('longGoals', longGoals);
            }
            renderGoals();
        }

        function renderGoals() {
            document.getElementById('short-display').textContent = shortGoals;
            document.getElementById('long-display').textContent = longGoals;
        }

        // Progress Charts
        function renderCharts() {
            const weekAgo = new Date();
            weekAgo.setDate(weekAgo.getDate() - 7);
            const weeklyLogs = studyLogs.filter(log => new Date(log.date) >= weekAgo);

            // Subject chart
            const subjectData = {};
            weeklyLogs.forEach(log => {
                subjectData[log.subject] = (subjectData[log.subject] || 0) + log.hours;
            });
            new Chart(document.getElementById('subject-chart'), {
                type: 'bar',
                data: {
                    labels: Object.keys(subjectData),
                    datasets: [{ label: 'Hours', data: Object.values(subjectData), backgroundColor: '#4CAF50' }]
                }
            });

            // Weekly chart (daily totals)
            const dailyData = {};
            weeklyLogs.forEach(log => {
                dailyData[log.date] = (dailyData[log.date] || 0) + log.hours;
            });
            new Chart(document.getElementById('weekly-chart'), {
                type: 'line',
                data: {
                    labels: Object.keys(dailyData),
                    datasets: [{ label: 'Daily Hours', data: Object.values(dailyData), borderColor: '#4CAF50' }]
                }
            });
        }
    </script>
</body>
</html>
