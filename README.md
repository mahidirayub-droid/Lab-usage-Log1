[lab-usage8.html](https://github.com/user-attachments/files/22136069/lab-usage8.html)
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Lab Usage Logger</title>
  <style>
    body {
      font-family: Arial, sans-serif;
      margin: 0; padding: 0;
      background: linear-gradient(to right, #1e3c72, #2a5298);
      color: #fff;
    }
    header {
      text-align: center;
      padding: 20px;
      background: rgba(0,0,0,0.4);
      font-size: 1.5em;
    }
    form {
      background: rgba(255,255,255,0.1);
      margin: 20px auto;
      padding: 20px;
      border-radius: 12px;
      width: 90%;
      max-width: 800px;
    }
    label { display:block; margin-top:10px; }
    input, select, textarea {
      width: 100%; padding: 6px;
      margin-top: 4px; border-radius: 6px;
      border: none; font-size: 1em;
    }
    button {
      margin-top: 15px;
      padding: 10px 20px;
      border: none; border-radius: 8px;
      background: #ff9800; color: #fff;
      font-size: 1em; cursor: pointer;
    }
    button:hover { background: #e68900; }
    table {
      width: 95%; margin: 20px auto;
      border-collapse: collapse;
      background: #fff; color:#000;
    }
    th, td {
      border: 1px solid #ccc; padding: 8px;
      text-align: center;
    }
    th { background: #2a5298; color:#fff; }
    .actions { text-align: center; margin: 20px; }
  </style>
</head>
<body>
  <header>Lab Usage Logger</header>

  <form id="labForm">
    <label>Date: <input type="date" id="date"></label>
    <label>Semester: <input type="text" id="semester"></label>
    <label>Course / Subjects: <input type="text" id="course"></label>
    <label>Credit Hours: <input type="number" id="creditHours" min="1" max="10"></label>

    <label>Devices Needed:
      <select id="devices" multiple>
        <option value="Cisco Equipment">Cisco Equipment</option>
        <option value="PCs">PCs</option>
        <option value="PC Assembly">PC Assembly</option>
        <option value="Wireless Router + Alfa">Wireless Router + Alfa</option>
        <option value="BYOD">BYOD</option>
      </select>
    </label>

    <label>Lab:
      <select id="lab">
        <option value="">-- Select Lab --</option>
        <option value="L15-012">L15-012</option>
        <option value="L15-009">L15-009</option>
        <option value="L12-001C">L12-001C</option>
        <option value="L12-002">L12-002</option>
        <option value="L04-003">L04-003</option>
        <option value="L04-004">L04-004</option>
        <option value="L10-007">L10-007</option>
        <option value="L10-005">L10-005</option>
        <option value="L10-004">L10-004</option>
      </select>
    </label>

    <label>Usage Ratio (%): <input type="number" id="usageRatio" min="0" max="100"></label>

    <label>TTOs in Charge: <input type="text" id="ttos"></label>
    <label>Class (January 2026): <input type="text" id="class" value="January 2026"></label>
    <label>Remark: <textarea id="remark"></textarea></label>

    <button type="button" onclick="addRecord()">Add Record</button>
  </form>

  <div class="actions">
    <button onclick="exportCSV()">Export CSV</button>
    <button onclick="importCSV()">Import CSV</button>
    <button onclick="clearData()">Clear All</button>
  </div>

  <!-- Table Rekod -->
  <h2 style="text-align:center;">Full Records</h2>
  <table id="recordsTable">
    <thead>
      <tr>
        <th>Date</th>
        <th>Semester</th>
        <th>Course / Subjects</th>
        <th>Credit Hours</th>
        <th>Devices Needed</th>
        <th>Lab</th>
        <th>Usage Ratio (%)</th>
        <th>TTOs in Charge</th>
        <th>Class Jan 2026</th>
        <th>Remark</th>
      </tr>
    </thead>
    <tbody></tbody>
  </table>

  <!-- Table Summary -->
  <h2 style="text-align:center;">Summary</h2>
  <table id="summaryTable">
    <thead>
      <tr>
        <th>Lab</th>
        <th>Usage</th>
      </tr>
    </thead>
    <tbody></tbody>
  </table>

  <script>
    const form = document.getElementById('labForm');
    const tableBody = document.querySelector('#recordsTable tbody');
    const summaryBody = document.querySelector('#summaryTable tbody');
    let records = JSON.parse(localStorage.getItem('labRecords') || '[]');

    document.getElementById('date').valueAsDate = new Date();

    function countLabUsage(labName) {
      return records.filter(r => r.lab === labName).length;
    }

    function updateSummary() {
      summaryBody.innerHTML = '';
      const labs = ["L15-012","L15-009","L12-001C","L12-002","L04-003","L04-004","L10-007","L10-005","L10-004"];
      labs.forEach(lab => {
        const tr = summaryBody.insertRow();
        tr.insertCell().textContent = `Lab ${lab}`;
        tr.insertCell().textContent = `${countLabUsage(lab)} / 13`;
      });
    }

    function renderTable() {
      tableBody.innerHTML = '';
      records.forEach(r => {
        const row = tableBody.insertRow();
        Object.values(r).forEach(val => {
          const cell = row.insertCell();
          cell.textContent = val;
        });
      });
      localStorage.setItem('labRecords', JSON.stringify(records));
      updateSummary();
    }

    function addRecord() {
      const date = document.getElementById('date').value;
      const semester = document.getElementById('semester').value;
      const course = document.getElementById('course').value;
      const creditHours = document.getElementById('creditHours').value;
      const devices = Array.from(document.getElementById('devices').selectedOptions).map(o => o.value).join(', ');
      const lab = document.getElementById('lab').value;
      const usageRatioVal = document.getElementById('usageRatio').value;
      const usageRatio = usageRatioVal ? usageRatioVal + '%' : '';
      const ttos = document.getElementById('ttos').value;
      const classJan = document.getElementById('class').value;
      const remark = document.getElementById('remark').value;

      // Rule: Max 13 classes per Lab
      if (lab && countLabUsage(lab) >= 13) {
        alert(`Lab ${lab} already has 13 classes. You cannot add more.`);
        return;
      }

      records.push({date, semester, course, creditHours, devices, lab, usageRatio, ttos, classJan, remark});
      renderTable();
      form.reset();
      document.getElementById('date').valueAsDate = new Date();
      document.getElementById('class').value = "January 2026";
    }

    function exportCSV() {
      let csv = '';
      const headers = ["Date","Semester","Course / Subjects","Credit Hours","Devices Needed","Lab","Usage Ratio","TTOs in Charge","Class Jan 2026","Remark"];
      csv += headers.join(',') + '\n';
      records.forEach(r => {
        csv += Object.values(r).map(val => `"${val}"`).join(',') + '\n';
      });
      const blob = new Blob([csv], {type:'text/csv'});
      const url = URL.createObjectURL(blob);
      const a = document.createElement('a');
      a.href = url;
      a.download = 'lab_records.csv';
      a.click();
      URL.revokeObjectURL(url);
    }

    function importCSV() {
      const input = document.createElement('input');
      input.type = 'file';
      input.accept = '.csv';
      input.onchange = e => {
        const file = e.target.files[0];
        const reader = new FileReader();
        reader.onload = event => {
          const lines = event.target.result.split('\n').slice(1);
          records = lines.filter(l => l.trim()).map(line => {
            const vals = line.split(',').map(v => v.replace(/(^"|"$)/g,''));
            const [date,semester,course,creditHours,devices,lab,usageRatio,ttos,classJan,remark] = vals;
            return {date,semester,course,creditHours,devices,lab,usageRatio,ttos,classJan,remark};
          });
          renderTable();
        };
        reader.readAsText(file);
      };
      input.click();
    }

    function clearData() {
      if (confirm("Are you sure to clear all records?")) {
        records = [];
        renderTable();
      }
    }

    renderTable();
  </script>
</body>
</html>
