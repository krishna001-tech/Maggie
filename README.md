const { createApp } = Vue;

const app = createApp({
    data() {
        return {
            selectedDate: new Date(2026, 5, 1),
            newTask: '',
            tasks: JSON.parse(localStorage.getItem('tasks')) || [],
            routineCompletions: JSON.parse(localStorage.getItem('routineCompletions')) || {},
            dailyRoutines: [
                {
                    time: '6:00 AM',
                    title: '🌅 MORNING RITUAL',
                    tasks: [
                        { icon: '📱', text: 'No phone – first hour of morning', completed: false },
                        { icon: '💧', text: 'Drink a full glass of H₂O (keep bottle handy)', completed: false },
                        { icon: '🧘', text: 'Meditate 5 mins', completed: false },
                        { icon: '📋', text: 'Read your goals out loud', completed: false }
                    ]
                },
                {
                    time: '7:00 AM',
                    title: '🏋️ MORNING WORKOUT',
                    tasks: [
                        { icon: '🏋️', text: 'Train (gym / bodyweight) — 4x/week', completed: false },
                        { icon: '🚫', text: 'No negative self-talk during workout', completed: false }
                    ]
                },
                {
                    time: '9:00 AM',
                    title: '🎯 DEEP WORK',
                    tasks: [
                        { icon: '✏️', text: 'Work on your chosen skill (content, editing…)', completed: false },
                        { icon: '📈', text: 'Learn personal finance / investing (15 mins)', completed: false }
                    ]
                },
                {
                    time: '12:00 PM',
                    title: '🥗 MIDDAY FUEL',
                    tasks: [
                        { icon: '🥦', text: 'No junk food (Sunday cheat allowed)', completed: false },
                        { icon: '💧', text: 'Drink H₂O with meal', completed: false }
                    ]
                },
                {
                    time: '2:00 PM',
                    title: '🤝 SOCIAL HOUR',
                    tasks: [
                        { icon: '📞', text: 'Call a long-distance friend', completed: false },
                        { icon: '🚫', text: 'Stop texting people who don\'t value you', completed: false },
                        { icon: '🤐', text: 'No gossiping today', completed: false },
                        { icon: '🛡️', text: 'Enforce one boundary', completed: false }
                    ]
                },
                {
                    time: '5:00 PM',
                    title: '🌇 AFTERNOON RESET',
                    tasks: [
                        { icon: '🧹', text: 'Keep room clean (5-min tidy)', completed: false },
                        { icon: '🗺️', text: 'One adventure / new experience this week', completed: false }
                    ]
                },
                {
                    time: '9:00 PM',
                    title: '🌙 NIGHT WIND-DOWN',
                    tasks: [
                        { icon: '📖', text: 'Read 10 pages (fiction)', completed: false },
                        { icon: '✍️', text: 'Journal 5–10 mins', completed: false },
                        { icon: '📵', text: 'No phone – 30 mins before bed', completed: false },
                        { icon: '😴', text: 'Sleep 7–8 hrs (non-negotiable)', completed: false }
                    ]
                }
            ]
        };
    },
    computed: {
        calendarDates() {
            const dates = [];
            const year = 2026;
            const month = 5; // June (0-indexed)
            
            // First day of month
            const firstDay = new Date(year, month, 1).getDay();
            
            // Days from previous month
            for (let i = firstDay - 1; i >= 0; i--) {
                dates.push(null);
            }
            
            // Days of current month
            const daysInMonth = new Date(year, month + 1, 0).getDate();
            for (let i = 1; i <= daysInMonth; i++) {
                dates.push(i);
            }
            
            // Days from next month
            const totalCells = dates.length;
            const remainingCells = 42 - totalCells;
            for (let i = 1; i <= remainingCells; i++) {
                dates.push(null);
            }
            
            return dates;
        },
        selectedDateFormatted() {
            const options = { weekday: 'long', year: 'numeric', month: 'long', day: 'numeric' };
            return this.selectedDate.toLocaleDateString('en-US', options);
        },
        tasksForSelectedDate() {
            const dateKey = this.getDateKey(this.selectedDate);
            return this.tasks.filter(task => task.date === dateKey);
        }
    },
    methods: {
        selectDate(date) {
            if (date) {
                this.selectedDate = new Date(2026, 5, date);
            }
        },
        isToday(date) {
            if (!date) return false;
            const today = new Date();
            return date === today.getDate() && 
                   this.selectedDate.getMonth() === today.getMonth() && 
                   this.selectedDate.getFullYear() === today.getFullYear();
        },
        hasTasksForDate(date) {
            if (!date) return false;
            const checkDate = new Date(2026, 5, date);
            const dateKey = this.getDateKey(checkDate);
            return this.tasks.some(task => task.date === dateKey);
        },
        getDateKey(date) {
            return date.toISOString().split('T')[0];
        },
        addTask() {
            if (this.newTask.trim()) {
                const task = {
                    id: Date.now(),
                    text: this.newTask,
                    completed: false,
                    date: this.getDateKey(this.selectedDate)
                };
                this.tasks.push(task);
                this.newTask = '';
                this.saveTasks();
            }
        },
        deleteTask(id) {
            this.tasks = this.tasks.filter(task => task.id !== id);
            this.saveTasks();
        },
        toggleTask(id) {
            const task = this.tasks.find(task => task.id === id);
            if (task) {
                task.completed = !task.completed;
                this.saveTasks();
            }
        },
        toggleRoutineTask(routineIndex, taskIndex) {
            const dateKey = this.getDateKey(this.selectedDate);
            if (!this.routineCompletions[dateKey]) {
                this.routineCompletions[dateKey] = {};
            }
            if (!this.routineCompletions[dateKey][routineIndex]) {
                this.routineCompletions[dateKey][routineIndex] = {};
            }
            
            const currentState = this.routineCompletions[dateKey][routineIndex][taskIndex];
            this.routineCompletions[dateKey][routineIndex][taskIndex] = !currentState;
            
            this.dailyRoutines[routineIndex].tasks[taskIndex].completed = 
                this.routineCompletions[dateKey][routineIndex][taskIndex];
            
            this.saveRoutineCompletions();
        },
        saveTasks() {
            localStorage.setItem('tasks', JSON.stringify(this.tasks));
        },
        saveRoutineCompletions() {
            localStorage.setItem('routineCompletions', JSON.stringify(this.routineCompletions));
        },
        loadRoutineCompletions() {
            const dateKey = this.getDateKey(this.selectedDate);
            if (this.routineCompletions[dateKey]) {
                this.dailyRoutines.forEach((routine, routineIndex) => {
                    routine.tasks.forEach((task, taskIndex) => {
                        if (this.routineCompletions[dateKey][routineIndex]) {
                            task.completed = this.routineCompletions[dateKey][routineIndex][taskIndex] || false;
                        }
                    });
                });
            } else {
                this.dailyRoutines.forEach(routine => {
                    routine.tasks.forEach(task => {
                        task.completed = false;
                    });
                });
            }
        }
    },
    watch: {
        selectedDate() {
            this.loadRoutineCompletions();
        }
    },
    mounted() {
        this.loadRoutineCompletions();
    }
});

app.mount('#app');
