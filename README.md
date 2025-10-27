<?php
// === DATABASETILKOBLING ===
$host = "localhost";
$user = "root";      // endre hvis Dokploy krever annen bruker
$pass = "";          // ev. passord
$dbname = "skole";   // databasenavn

$conn = new mysqli($host, $user, $pass, $dbname);
if ($conn->connect_error) {
  die("Tilkoblingsfeil: " . $conn->connect_error);
}

// === FUNKSJON FOR SLETTING ===
if (isset($_GET['slett_klasse'])) {
  $kode = $_GET['slett_klasse'];
  $conn->query("DELETE FROM klasse WHERE klassekode='$kode'");
  header("Location: index.php?vis=klasse");
  exit;
}

if (isset($_GET['slett_student'])) {
  $bruker = $_GET['slett_student'];
  $conn->query("DELETE FROM student WHERE brukernavn='$bruker'");
  header("Location: index.php?vis=student");
  exit;
}

// === FUNKSJON FOR REGISTRERING ===
if (isset($_POST['lagre_klasse'])) {
  $kode = $_POST['kode'];
  $navn = $_POST['navn'];
  $studium = $_POST['studium'];
  $conn->query("INSERT INTO klasse VALUES ('$kode', '$navn', '$studium')");
  header("Location: index.php?vis=klasse");
  exit;
}

if (isset($_POST['lagre_student'])) {
  $bruker = $_POST['bruker'];
  $fornavn = $_POST['fornavn'];
  $etternavn = $_POST['etternavn'];
  $klassekode = $_POST['klassekode'];
  $conn->query("INSERT INTO student VALUES ('$bruker', '$fornavn', '$etternavn', '$klassekode')");
  header("Location: index.php?vis=student");
  exit;
}
?>

<!DOCTYPE html>
<html lang="no">
<head>
  <meta charset="UTF-8">
  <title>Vedlikeholdssystem</title>
  <style>
    body { font-family: Arial, sans-serif; margin: 30px; }
    table { border-collapse: collapse; margin-top: 10px; }
    th, td { border: 1px solid #999; padding: 6px 10px; }
    h1 { margin-bottom: 10px; }
    a { text-decoration: none; color: #0055cc; }
    a:hover { text-decoration: underline; }
    form { margin-top: 15px; }
    input, select { margin: 3px 0; padding: 4px; }
  </style>
</head>
<body>

<h1>Vedlikeholdssystem</h1>
<ul>
  <li><a href="index.php?vis=klasse">Administrer klasser</a></li>
  <li><a href="index.php?vis=student">Administrer studenter</a></li>
</ul>

<hr>

<?php
// === VIS KLASSE-DEL ===
if (isset($_GET['vis']) && $_GET['vis'] == 'klasse') {
  echo "<h2>Klasser</h2>";

  $result = $conn->query("SELECT * FROM klasse");
  echo "<table><tr><th>Kode</th><th>Navn</th><th>Studiumkode</th><th>Slett</th></tr>";
  while ($row = $result->fetch_assoc()) {
    echo "<tr>
      <td>{$row['klassekode']}</td>
      <td>{$row['klassenavn']}</td>
      <td>{$row['studiumkode']}</td>
      <td><a href='?slett_klasse={$row['klassekode']}'>Slett</a></td>
    </tr>";
  }
  echo "</table>";

  echo "
  <h3>Registrer ny klasse</h3>
  <form method='POST'>
    Kode: <input type='text' name='kode' required><br>
    Navn: <input type='text' name='navn' required><br>
    Studiumkode: <input type='text' name='studium' required><br>
    <input type='submit' name='lagre_klasse' value='Lagre'>
  </form>
  ";
}

// === VIS STUDENT-DEL ===
elseif (isset($_GET['vis']) && $_GET['vis'] == 'student') {
  echo "<h2>Studenter</h2>";

  $result = $conn->query("SELECT * FROM student");
  echo "<table><tr><th>Brukernavn</th><th>Fornavn</th><th>Etternavn</th><th>Klasse</th><th>Slett</th></tr>";
  while ($row = $result->fetch_assoc()) {
    echo "<tr>
      <td>{$row['brukernavn']}</td>
      <td>{$row['fornavn']}</td>
      <td>{$row['etternavn']}</td>
      <td>{$row['klassekode']}</td>
      <td><a href='?slett_student={$row['brukernavn']}'>Slett</a></td>
    </tr>";
  }
  echo "</table>";

  // Hent klasseliste
  $res = $conn->query("SELECT klassekode FROM klasse");
  echo "
  <h3>Registrer ny student</h3>
  <form method='POST'>
    Brukernavn: <input type='text' name='bruker' required><br>
    Fornavn: <input type='text' name='fornavn' required><br>
    Etternavn: <input type='text' name='etternavn' required><br>
    Klassekode: <select name='klassekode'>";
  while ($r = $res->fetch_assoc()) {
    echo "<option value='{$r['klassekode']}'>{$r['klassekode']}</option>";
  }
  echo "</select><br>
    <input type='submit' name='lagre_student' value='Lagre'>
  </form>
  ";
}

// === STANDARD FORSIDE ===
else {
  echo "<p>Velg hva du vil administrere fra menyen over.</p>";
}
?>

</body>
</html>
