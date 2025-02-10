<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>To-Do List</title>
  <style>
    body {
      font-family: Arial, sans-serif;
      background-color: #f4f4f4;
      margin: 0;
      padding: 20px;
      transition: background 0.3s, color 0.3s;
    }
    .container {
      max-width: 500px;
      margin: auto;
      background: white;
      padding: 20px;
      border-radius: 10px;
      box-shadow: 0 0 10px rgba(0, 0, 0, 0.1);
    }
    .header {
      display: flex;
      justify-content: space-between;
      align-items: center;
    }
    .theme-toggle {
      cursor: pointer;
      font-size: 20px;
    }
    .task-input-container {
      display: flex;
      flex-wrap: wrap;
      gap: 10px;
      margin: 20px 0;
    }
    .task-input-container input[type="text"] {
      flex: 1;
      padding: 10px;
      border: 1px solid #ccc;
      border-radius: 5px;
    }
    .task-input-container input[type="date"] {
      padding: 10px;
      border: 1px solid #ccc;
      border-radius: 5px;
    }
    .task-input-container button {
      padding: 10px 15px;
      border: none;
      background-color: #28a745;
      color: white;
      border-radius: 5px;
      cursor: pointer;
    }
    .task-input-container button:hover {
      background-color: #218838;
    }
    .task-section {
      margin-bottom: 20px;
    }
    .task-section h3 {
      margin-bottom: 10px;
    }
    ul {
      list-style: none;
      padding: 0;
    }
    li {
      padding: 12px;
      background: #fff;
      border-radius: 5px;
      margin-bottom: 8px;
      display: flex;
      justify-content: space-between;
      align-items: center;
      border: 1px solid #ccc;
      cursor: pointer;
    }
    li.completed {
      text-decoration: line-through;
      color: gray;
    }
    .task-buttons {
      display: flex;
      gap: 8px;
    }
    .task-buttons button {
      margin-left: 5px;
      border: none;
      background: none;
      cursor: pointer;
      font-size: 16px;
    }
    #clearCompleted {
      padding: 10px 15px;
      background-color: #dc3545;
      color: white;
      border: none;
      border-radius: 5px;
      cursor: pointer;
    }
    #clearCompleted:hover {
      background-color: #c82333;
    }
    /* Dark Mode Styles */
    body.dark-mode {
      background-color: #222;
      color: #fff;
    }
    body.dark-mode .container {
      background: #333;
      box-shadow: 0 0 10px rgba(255, 255, 255, 0.1);
    }
    body.dark-mode input,
    body.dark-mode button {
      border-color: #555;
    }
  </style>
</head>
<body>
  <div class="container">
    <div class="header">
      <h1>To-Do List</h1>
      <span class="theme-toggle">🌙</span>
    </div>

    <div class="task-input-container">
      <input type="text" id="taskInput" placeholder="New task..." />
      <!-- Due date input; most browsers display a date picker -->
      <input type="date" id="dueDate" />
      <button id="addTaskButton">Add</button>
    </div>

    <div class="task-section">
      <h3>Pending Tasks</h3>
      <ul id="pendingTasks"></ul>
    </div>

    <div class="task-section">
      <h3>Completed Tasks</h3>
      <ul id="completedTasks"></ul>
    </div>

    <button id="clearCompleted">Clear Completed Tasks</button>
  </div>

  <script>
    const taskInput = document.getElementById('taskInput');
    const dueDateInput = document.getElementById('dueDate');
    const addTaskButton = document.getElementById('addTaskButton');
    const pendingTasksList = document.getElementById('pendingTasks');
    const completedTasksList = document.getElementById('completedTasks');
    const clearCompletedButton = document.getElementById('clearCompleted');
    const themeToggle = document.querySelector('.theme-toggle');

    // Retrieve saved tasks from localStorage or start with an empty array
    let tasks = JSON.parse(localStorage.getItem('tasks')) || [];

    // Save tasks to localStorage
    function saveTasks() {
      localStorage.setItem('tasks', JSON.stringify(tasks));
    }

    // Render the tasks into their respective sections
    function renderTasks() {
      pendingTasksList.innerHTML = '';
      completedTasksList.innerHTML = '';

      tasks.forEach((task, index) => {
        const li = document.createElement('li');
        // Display task text and due date (formatted as dd-mm-yyyy if provided)
        li.textContent = task.text + (task.dueDate ? ' - ' + formatDate(task.dueDate) : '');

        // Create buttons for edit and delete
        const buttonsDiv = document.createElement('div');
        buttonsDiv.className = 'task-buttons';

        const editButton = document.createElement('button');
        editButton.textContent = '✏️';
        editButton.addEventListener('click', function (e) {
          e.stopPropagation();
          editTask(index);
        });

        const deleteButton = document.createElement('button');
        deleteButton.textContent = '🗑️';
        deleteButton.addEventListener('click', function (e) {
          e.stopPropagation();
          deleteTask(index);
        });

        buttonsDiv.appendChild(editButton);
        buttonsDiv.appendChild(deleteButton);
        li.appendChild(buttonsDiv);

        // Clicking the task toggles its completion status
        li.addEventListener('click', () => toggleComplete(index));

        if (task.completed) {
          li.classList.add('completed');
          completedTasksList.appendChild(li);
        } else {
          pendingTasksList.appendChild(li);
        }
      });

      saveTasks();
    }

    // Format date from "yyyy-mm-dd" to "dd-mm-yyyy"
    function formatDate(dateString) {
      const parts = dateString.split('-');
      if (parts.length === 3) {
        return parts[2] + '-' + parts[1] + '-' + parts[0];
      }
      return dateString;
    }

    // Add a new task with text and an optional due date
    function addTask() {
      const text = taskInput.value.trim();
      const dueDate = dueDateInput.value; // Format: yyyy-mm-dd
      if (!text) return;
      tasks.push({ text, dueDate, completed: false });
      taskInput.value = '';
      dueDateInput.value = '';
      renderTasks();
    }

    // Edit an existing task
    function editTask(index) {
      const newText = prompt('Edit task:', tasks[index].text);
      if (newText !== null && newText.trim() !== '') {
        tasks[index].text = newText.trim();
        renderTasks();
      }
    }

    // Delete a task
    function deleteTask(index) {
      tasks.splice(index, 1);
      renderTasks();
    }

    // Toggle a task's completed status
    function toggleComplete(index) {
      tasks[index].completed = !tasks[index].completed;
      renderTasks();
    }

     // Clear all tasks that have been marked as completed
     function clearCompletedTasks() {
      tasks = tasks.filter(task => !task.completed);
      renderTasks();
    }

    // Event listeners for the Add button, Enter key, Clear Completed, and Theme toggle
    addTaskButton.addEventListener('click', addTask);
    taskInput.addEventListener('keypress', e => {
      if (e.key === 'Enter') addTask();
    });
    clearCompletedButton.addEventListener('click', clearCompletedTasks);
    themeToggle.addEventListener('click', () => {
      document.body.classList.toggle('dark-mode');
      themeToggle.textContent = document.body.classList.contains('dark-mode') ? '☀️' : '🌙';
    });

    // Initial render of tasks
    renderTasks();
  </script>
</body>
</html>
