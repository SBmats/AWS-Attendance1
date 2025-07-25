<!DOCTYPE html>
<html>
  <head>
    <title>Student Exam Portal</title>
    <style>
      body {
        font-family: "Segoe UI", sans-serif;
        background: linear-gradient(135deg, #a18cd1, #fbc2eb);
        margin: 0;
        padding: 40px;
        backdrop-filter: blur(10px);
      }
      .glass-container {
        max-width: 800px;
        margin: auto;
        background: rgba(255, 255, 255, 0.15);
        padding: 30px;
        border-radius: 16px;
        box-shadow: 0 8px 32px 0 rgba(31, 38, 135, 0.37);
        backdrop-filter: blur(8px);
        border: 1px solid rgba(255, 255, 255, 0.18);
        color: #fff;
      }
      h2,
      h3 {
        text-align: center;
        color: #fff;
      }
      input,
      select,
      button {
        width: 100%;
        padding: 10px;
        margin: 10px 0;
        border-radius: 10px;
        border: none;
        font-size: 16px;
      }
      button {
        background: #8e44ad;
        color: #fff;
        cursor: pointer;
        font-weight: bold;
      }
      button:hover {
        background: #6c3483;
      }
      .hidden {
        display: none;
      }
      table {
        width: 100%;
        border-collapse: collapse;
        background: rgba(255, 255, 255, 0.1);
      }
      th,
      td {
        border: 1px solid rgba(255, 255, 255, 0.3);
        padding: 10px;
        text-align: center;
        color: #fff;
      }
      #questionBox label {
        display: block;
        padding: 10px;
        background: rgba(255, 255, 255, 0.2);
        margin-bottom: 8px;
        border-radius: 10px;
        cursor: pointer;
        transition: background 0.3s ease;
      }
      #questionBox input[type="radio"] {
        display: none;
      }
      .selected {
        background: rgba(138, 43, 226, 0.5) !important;
        border: 2px solid #fff;
      }
      .timer {
        width: 100px;
        height: 100px;
        border: 6px solid #fff;
        border-radius: 50%;
        display: flex;
        align-items: center;
        justify-content: center;
        margin: 20px auto;
        font-size: 24px;
        background: rgba(255, 255, 255, 0.2);
      }
    </style>
  </head>
  <body>
    <div class="glass-container" id="loginSection">
      <h2>Student Login</h2>
      <form id="loginForm">
        <input type="text" id="name" placeholder="Name" required />
        <input type="text" id="roll" placeholder="Roll Number" required />
        <input type="text" id="branch" placeholder="Branch" required />
        <input type="tel" id="phone" placeholder="Phone Number" required />
        <select id="subject" required>
          <option value="">Select Subject</option>
          <option value="Maths">Maths</option>
          <option value="Science">Science</option>
          <option value="GK">GK</option>
        </select>
        <button type="submit">Login</button>
      </form>
      <div id="pastResults" class="hidden">
        <button id="startExamBtn">Start Exam</button>
        <h3>Past Results</h3>
        <div id="studentInfo"></div>
        <table>
          <thead>
            <tr>
              <th>Date</th>
              <th>Score</th>
              <th>Download</th>
            </tr>
          </thead>
          <tbody id="resultTableBody"></tbody>
        </table>
      </div>
    </div>
    <div class="glass-container hidden" id="examSection">
      <h2>Online Exam</h2>
      <div class="timer" id="timer">30</div>
      <div id="questionBox"></div>
      <button onclick="nextQuestion()">Next</button>
    </div>
    <div class="glass-container hidden" id="resultSection">
      <h2>Exam Result</h2>
      <div id="resultBox"></div>
      <button onclick="downloadExcel()">Download Result (CSV)</button>
      <button onclick="downloadPDF()">Download Q&A PDF</button>
    </div>

    <script src="https://cdnjs.cloudflare.com/ajax/libs/jspdf/2.5.1/jspdf.umd.min.js"></script>
    <script>
      let student = {}, current = 0, answers = [], time = 30, timerId;
      let questions = [], selectedSubject = "";

      const fixedQuestions = [
        { q: "What is 2+2?", options: ["3", "4", "5", "6"], answer: 1 },
        { q: "Capital of India?", options: ["Mumbai", "Delhi", "Kolkata", "Chennai"], answer: 1 },
        { q: "Color of sky?", options: ["Red", "Blue", "Green", "Yellow"], answer: 1 }
      ];
      const fillerQuestions = Array(47).fill().map((_, i) => ({
        q: Question ${i+4}: ${i+4} + ${i+5} = ?,
        options: [${i+9}, ${i+10}, ${i+11}, ${i+12}],
        answer: 0
      }));
      questions = [...fixedQuestions, ...fillerQuestions];

      document.getElementById("loginForm").onsubmit = function(e) {
        e.preventDefault();
        const roll = document.getElementById("roll").value;
        selectedSubject = document.getElementById("subject").value;
        student = {
          name: document.getElementById("name").value,
          roll,
          branch: document.getElementById("branch").value,
          phone: document.getElementById("phone").value,
          subject: selectedSubject
        };
        showPastResults(roll);
        document.getElementById("loginForm").classList.add("hidden");
        document.getElementById("pastResults").classList.remove("hidden");
        document.getElementById("studentInfo").innerHTML =
          <b>Name:</b> ${student.name}<br><b>Roll:</b> ${roll}<br><b>Phone:</b> ${student.phone};
      };

      document.getElementById("startExamBtn").onclick = () => {
        document.getElementById("loginSection").classList.add("hidden");
        document.getElementById("examSection").classList.remove("hidden");
        loadQuestion();
      };

      function loadQuestion() {
        if (current >= questions.length) return finishExam();
        const q = questions[current];
        document.getElementById("questionBox").innerHTML = `
          <p><strong>${current+1}. ${q.q}</strong></p>
          ${q.options.map((opt, i) =>
            <label onclick="selectAnswer(this, ${i})"><input type='radio' name='ans' value='${i}'> ${opt}</label>
          ).join('')}`;
        time = 30;
        document.getElementById("timer").innerText = time;
        timerId = setInterval(() => {
          time--;
          document.getElementById("timer").innerText = time;
          if (time <= 0) nextQuestion();
        }, 1000);
      }

      function selectAnswer(label, val) {
        const labels = document.querySelectorAll('#questionBox label');
        labels.forEach(l => l.classList.remove("selected"));
        label.classList.add("selected");
      }

      function nextQuestion() {
        clearInterval(timerId);
        const selected = document.querySelector('input[name="ans"]:checked');
        answers.push(selected ? parseInt(selected.value) : -1);
        current++;
        loadQuestion();
      }

      function finishExam() {
        clearInterval(timerId);
        document.getElementById("examSection").classList.add("hidden");
        document.getElementById("resultSection").classList.remove("hidden");
        const score = answers.filter((a, i) => a === questions[i].answer).length;
        const result = { date: new Date().toLocaleString(), score, answers, ...student };
        const key = results_${student.roll};
        const existing = JSON.parse(localStorage.getItem(key)) || [];
        existing.push(result);
        localStorage.setItem(key, JSON.stringify(existing));
        document.getElementById("resultBox").innerHTML =
          Name: ${student.name}<br>Roll: ${student.roll}<br>Subject: ${student.subject}<br>Score: ${score} / 50;

        window.downloadExcel = function () {
          const csv = Name,Roll,Branch,Phone,Subject,Score,Date\n${student.name},${student.roll},${student.branch},${student.phone},${student.subject},${score},${result.date};
          const blob = new Blob([csv], {type: "text/csv"});
          const link = document.createElement("a");
          link.href = URL.createObjectURL(blob);
          link.download = ${student.roll}_result.csv;
          link.click();
        };

        window.downloadPDF = function () {
          const { jsPDF } = window.jspdf;
          const doc = new jsPDF();
          let y = 10;
          doc.text(Exam Report - ${student.name}, 10, y); y += 10;
          questions.forEach((q, i) => {
            if (y > 270) { doc.addPage(); y = 10; }
            doc.text(${i+1}. ${q.q}, 10, y); y += 6;
            doc.text(Your Answer: ${answers[i] >= 0 ? q.options[answers[i]] : 'No Answer'}, 10, y); y += 6;
            doc.text(Correct: ${q.options[q.answer]}, 10, y); y += 8;
          });
          doc.save(${student.roll}_QA_Report.pdf);
        };
      }

      function showPastResults(roll) {
        const results = JSON.parse(localStorage.getItem(results_${roll})) || [];
        const body = document.getElementById("resultTableBody");
        body.innerHTML = '';
        results.forEach((res, i) => {
          const row = document.createElement("tr");
          row.innerHTML = `
            <td>${res.date}</td>
            <td>${res.score} / 50</td>
            <td><button onclick="downloadPastResult('${roll}', ${i})">Download</button></td>`;
          body.appendChild(row);
        });
      }

      function downloadPastResult(roll, index) {
        const results = JSON.parse(localStorage.getItem(results_${roll})) || [];
        const r = results[index];
        const csv = Name,Roll,Branch,Phone,Subject,Score,Date\n${r.name},${r.roll},${r.branch},${r.phone},${r.subject},${r.score},${r.date};
        const blob = new Blob([csv], {type: "text/csv"});
        const link = document.createElement("a");
        link.href = URL.createObjectURL(blob);
        link.download = ${r.roll}_result_${index+1}.csv;
        link.click();
      }

      document.addEventListener("visibilitychange", () => {
        if (document.hidden) {
          alert("Tab switch or phone call detected. Exam ended.");
          finishExam();
        }
      });
    </script>
  </body>
</html>
