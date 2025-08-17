#resultmaker.anmol 

<publised by anmol singh>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <title>Student Result Manager by anmol</title>
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <style>
    body {
      font-family: Arial, sans-serif;
      margin: 20px;
      background: #f9f9fb;
      position: relative;
    }

    h2 {
      color: #2d2d6b;
    }

    .grid-container {
      display: grid;
      grid-template-columns: repeat(auto-fit, minmax(150px, 1fr));
      gap: 10px;
      margin-bottom: 10px;
    }

    .subject-box {
      background: white;
      padding: 10px;
      border-radius: 6px;
      box-shadow: 0 0 3px rgba(0,0,0,0.1);
    }

    label {
      display: block;
      font-size: 14px;
      margin-bottom: 5px;
    }

    input {
      padding: 6px;
      width: 100%;
      font-size: 14px;
    }

    button {
      padding: 10px;
      margin-top: 10px;
      font-weight: bold;
      background-color: #2d2d6b;
      color: white;
      border: none;
      border-radius: 5px;
      cursor: pointer;
    }

    button:hover {
      background-color: #1a1a40;
    }

    table {
      width: 100%;
      border-collapse: collapse;
      margin-top: 20px;
    }

    th, td {
      border: 1px solid #ccc;
      padding: 8px;
      text-align: center;
    }

    .modal {
      display: none;
      position: fixed;
      z-index: 100;
      left: 0;
      top: 0;
      width: 100%;
      height: 100%;
      overflow: auto;
      background: rgba(0,0,0,0.4);
    }

    .modal-content {
      background: white;
      margin: 10% auto;
      padding: 20px;
      border-radius: 10px;
      width: 90%;
      max-width: 500px;
    }

    .subject-row {
      display: flex;
      gap: 10px;
      margin-bottom: 10px;
    }

    .subject-row input {
      flex: 1;
    }

    .remove-btn {
      background-color: red;
      color: white;
      border: none;
      padding: 4px 8px;
      cursor: pointer;
    }

    .watermark {
      position: fixed;
      top: 50%;
      left: 50%;
      transform: translate(-50%, -50%) rotate(-30deg);
      font-size: 50px;
      color: rgba(0, 0, 0, 0.05);
      user-select: none;
      z-index: 0;
      white-space: nowrap;
    }

    @media (max-width: 600px) {
      .subject-row {
        flex-direction: column;
      }
    }
  </style>
</head>
<body>

<h2>📘 Student Result Manager</h2>

<div>
  <input type="text" id="studentName" placeholder="Student Name" />
</div>

<div id="subjectsContainer" class="grid-container"></div>

<button onclick="addStudent()">➕ Add Student</button>
<button onclick="openModal()">⚙️ Customize Subjects</button>
<button onclick="exportToExcel()">📤 Export to Excel</button>

<table>
  <thead>
    <tr>
      <th>Name</th>
      <th>Total Obtained</th>
      <th>Total Marks</th>
      <th>Percentage</th>
      <th>Grade</th>
      <th>Result</th>
    </tr>
  </thead>
  <tbody id="resultBody"></tbody>
</table>

<!-- Watermark -->
<div class="watermark">ANMOL SINGH</div>

<!-- Modal -->
<div id="subjectModal" class="modal">
  <div class="modal-content">
    <h3>Customize Subjects</h3>
    <div id="modalSubjects"></div>
    <button onclick="addModalSubject()">+ Add Subject</button>
    <button onclick="saveSubjects()">✅ Save</button>
    <button onclick="closeModal()">❌ Close</button>
  </div>
</div>

<!-- SheetJS -->
<script src="https://cdn.sheetjs.com/xlsx-0.19.3/package/dist/xlsx.full.min.js"></script>

<script>
  let subjects = [
    { name: "Hindi", total: 100 },
    { name: "English", total: 100 },
    { name: "Maths", total: 100 },
    { name: "Science", total: 100 },
    { name: "SST", total: 100 }
  ];

  const students = [];

  function renderSubjects() {
    const container = document.getElementById("subjectsContainer");
    container.innerHTML = "";
    subjects.forEach((subject, index) => {
      const div = document.createElement("div");
      div.className = "subject-box";
      div.innerHTML = `
        <label>${subject.name} (out of ${subject.total})</label>
        <input type="number" id="subject-${index}" placeholder="Obtained Marks" min="0" max="${subject.total}" />
      `;
      container.appendChild(div);
    });
  }

  function openModal() {
    document.getElementById("subjectModal").style.display = "block";
    const modalSubjects = document.getElementById("modalSubjects");
    modalSubjects.innerHTML = "";
    subjects.forEach((subject, index) => {
      const row = document.createElement("div");
      row.className = "subject-row";
      row.innerHTML = `
        <input type="text" value="${subject.name}" placeholder="Subject Name" />
        <input type="number" value="${subject.total}" placeholder="Total Marks" min="1" />
        <button class="remove-btn" onclick="removeModalSubject(${index})">X</button>
      `;
      modalSubjects.appendChild(row);
    });
  }

  function addModalSubject() {
    subjects.push({ name: "New Subject", total: 100 });
    openModal();
  }

  function removeModalSubject(index) {
    subjects.splice(index, 1);
    openModal();
  }

  function saveSubjects() {
    const modalSubjects = document.getElementById("modalSubjects").children;
    subjects = [];
    for (let row of modalSubjects) {
      const name = row.children[0].value.trim() || "Subject";
      const total = parseInt(row.children[1].value) || 100;
      subjects.push({ name, total });
    }
    closeModal();
    renderSubjects();
  }

  function closeModal() {
    document.getElementById("subjectModal").style.display = "none";
  }

  function getGrade(percentage) {
    if (percentage >= 90) return "A+";
    if (percentage >= 80) return "A";
    if (percentage >= 70) return "B";
    if (percentage >= 60) return "C";
    if (percentage >= 50) return "D";
    return "F";
  }

  function addStudent() {
    const name = document.getElementById("studentName").value.trim();
    if (!name) return alert("Enter student name.");

    let totalMarks = 0;
    let totalObtained = 0;
    let failed = false;

    subjects.forEach((subject, index) => {
      const val = parseFloat(document.getElementById(`subject-${index}`).value);
      if (isNaN(val) || val < 0 || val > subject.total) {
        alert(`Enter valid marks for ${subject.name}`);
        return;
      }
      totalMarks += subject.total;
      totalObtained += val;
      if (val < subject.total * 0.33) failed = true;
    });

    const percent = ((totalObtained / totalMarks) * 100).toFixed(2);
    const grade = getGrade(percent);
    const result = failed ? "Fail" : "Pass";

    students.push({
      Name: name,
      Obtained: totalObtained,
      Total: totalMarks,
      Percentage: percent + "%",
      Grade: grade,
      Result: result
    });

    renderTable();
    document.getElementById("studentName").value = "";
    subjects.forEach((_, i) => document.getElementById(`subject-${i}`).value = "");
  }

  function renderTable() {
    const tbody = document.getElementById("resultBody");
    tbody.innerHTML = "";
    students.forEach(s => {
      const tr = document.createElement("tr");
      tr.innerHTML = `
        <td>${s.Name}</td>
        <td>${s.Obtained}</td>
        <td>${s.Total}</td>
        <td>${s.Percentage}</td>
        <td>${s.Grade}</td>
        <td>${s.Result}</td>
      `;
      tbody.appendChild(tr);
    });
  }

  function exportToExcel() {
    if (students.length === 0) return alert("No data to export.");
    const ws = XLSX.utils.json_to_sheet(students);
    const wb = XLSX.utils.book_new();
    XLSX.utils.book_append_sheet(wb, ws, "Results");
    XLSX.writeFile(wb, "Student_Results.xlsx");
  }

  renderSubjects();
</script>

</body>
</html>
