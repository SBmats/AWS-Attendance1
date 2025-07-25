<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    
  </head>
  <body>
    <h1>🎉 Selected Students List</h1>

    <div class="search-box">
      <input
        type="text"
        id="searchInput"
        placeholder="Search by name, roll or company..."
        onkeyup="filterStudents()"
      />
    </div>

    <div class="students" id="studentsList">
      <div class="card">
        <h2>👤 John Doe</h2>
        <p>🎓 Roll No: 21CS045</p>
        <p>🧪 Branch: CSE</p>
        <p>🏢 Company: Google</p>
      </div>

      <div class="card">
        <h2>👤 Priya Sharma</h2>
        <p>🎓 Roll No: 21EC011</p>
        <p>🧪 Branch: ECE</p>
        <p>🏢 Company: Infosys</p>
      </div>

      <div class="card">
        <h2>👤 Rahul Kumar</h2>
        <p>🎓 Roll No: 21ME022</p>
        <p>🧪 Branch: Mech</p>
        <p>🏢 Company: TCS</p>
      </div>

      <!-- Add more cards as needed -->
    </div>

    <script>
      function filterStudents() {
        const input = document
          .getElementById("searchInput")
          .value.toLowerCase();
        const cards = document.querySelectorAll(".card");
        cards.forEach((card) => {
          const text = card.innerText.toLowerCase();
          card.style.display = text.includes(input) ? "block" : "none";
        });
      }
    </script>
  </body>
</html>
