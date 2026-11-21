<!DOCTYPE html>
<html lang="pl">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Mój Blog (Firestore)</title>
  <style>
    :root {
      --bg: #0f0f13;
      --card: #1a1a22;
      --border: #2c2c3a;
      --text: #f1f1f5;
      --muted: #9a9aa8;
      --accent: #6c5ce7;
      --accent-hover: #5849c4;
      --danger: #e74c3c;
      --admin: #f1c40f;
    }
    * { box-sizing: border-box; margin: 0; padding: 0; font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, sans-serif; }
    body { background: var(--bg); color: var(--text); padding-bottom: 40px; }

    header {
      background: var(--card);
      border-bottom: 1px solid var(--border);
      padding: 15px 20px;
      position: sticky;
      top: 0;
      z-index: 100;
      display: flex;
      justify-content: space-between;
      align-items: center;
      flex-wrap: wrap;
      gap: 10px;
    }
    header h1 { font-size: 1.2rem; letter-spacing: 0.5px; }
    .badge-admin { background: var(--admin); color: #000; font-size: 0.7rem; font-weight: 800; padding: 2px 6px; border-radius: 4px; margin-left: 6px; }

    .container { max-width: 650px; margin: 20px auto; padding: 0 15px; }

    .card {
      background: var(--card);
      border: 1px solid var(--border);
      border-radius: 12px;
      padding: 18px;
      margin-bottom: 20px;
    }

    input, textarea {
      width: 100%;
      background: #121218;
      border: 1px solid var(--border);
      color: var(--text);
      padding: 10px 12px;
      border-radius: 8px;
      margin-bottom: 10px;
      font-size: 0.95rem;
    }
    input:focus, textarea:focus { outline: none; border-color: var(--accent); }

    button {
      background: var(--accent);
      color: #fff;
      border: none;
      padding: 9px 15px;
      border-radius: 8px;
      font-weight: 600;
      cursor: pointer;
      font-size: 0.9rem;
    }
    button:hover { background: var(--accent-hover); }
    button.secondary { background: #2c2c3a; }
    button.danger { background: var(--danger); padding: 5px 10px; font-size: 0.8rem; }
    button.small { padding: 6px 10px; font-size: 0.8rem; }

    .auth-tabs { display: flex; gap: 10px; margin-bottom: 12px; flex-wrap: wrap; }
    .hidden { display: none !important; }

    .post-header { display: flex; justify-content: space-between; align-items: center; margin-bottom: 10px; }
    .post-author { font-weight: 700; color: var(--admin); font-size: 0.95rem; }
    .post-date { color: var(--muted); font-size: 0.78rem; }
    .post-title { font-size: 1.25rem; font-weight: 700; margin-bottom: 8px; }
    .post-content { line-height: 1.5; white-space: pre-wrap; margin-bottom: 12px; color: #e2e2ea; }
    .post-img { width: 100%; max-height: 450px; object-fit: cover; border-radius: 8px; margin-bottom: 12px; border: 1px solid var(--border); }

    .post-note {
      background: rgba(241, 196, 15, 0.1);
      border-left: 3px solid var(--admin);
      padding: 10px 12px;
      border-radius: 0 8px 8px 0;
      margin-bottom: 12px;
      font-size: 0.9rem;
    }
    .post-note strong { color: var(--admin); display: block; font-size: 0.75rem; text-transform: uppercase; margin-bottom: 3px; }

    .post-actions { display: flex; gap: 10px; align-items: center; border-top: 1px solid var(--border); padding-top: 12px; margin-top: 10px; flex-wrap: wrap; }
    .like-btn { background: #252532; display: flex; align-items: center; gap: 6px; }
    .like-btn.liked { background: #e84393; }

    .comments-section { margin-top: 14px; padding-top: 12px; border-top: 1px solid var(--border); }
    .comment { background: #13131a; padding: 8px 12px; border-radius: 8px; margin-bottom: 8px; font-size: 0.88rem; }
    .comment-head { display: flex; justify-content: space-between; margin-bottom: 3px; font-size: 0.78rem; color: var(--muted); }
    .comment-author { font-weight: 700; color: #a29bfe; }
    .comment-form { display: flex; gap: 8px; margin-top: 8px; }
    .comment-form input { margin-bottom: 0; }

    .admin-note-box { margin-top: 10px; display: flex; gap: 8px; }
    .admin-note-box input { margin-bottom: 0; }
  </style>
</head>
<body>

  <header>
    <h1>Mój Blog <span id="headerBadge" class="badge-admin hidden">ADMIN</span></h1>
    <div id="userBar">
      <button class="small" id="openAuthBtn">Zaloguj / Rejestracja</button>
    </div>
  </header>

  <div class="container">

    <!-- LOGOWANIE / REJESTRACJA -->
    <div id="authBox" class="card hidden">
      <div class="auth-tabs">
        <button class="small" id="tabLoginBtn">Logowanie</button>
        <button class="small secondary" id="tabRegisterBtn">Załóż konto (bez maila)</button>
        <button class="small secondary" style="margin-left:auto;" id="closeAuthBtn">✕</button>
      </div>

      <div id="loginForm">
        <input type="text" id="loginUser" placeholder="Login">
        <input type="password" id="loginPass" placeholder="Hasło">
        <button id="loginSubmitBtn">Zaloguj się</button>
      </div>

      <div id="registerForm" class="hidden">
        <input type="text" id="regUser" placeholder="Wybierz swój nick (login)">
        <input type="password" id="regPass" placeholder="Ustaw hasło">
        <button id="registerSubmitBtn">Utwórz konto</button>
      </div>
    </div>

    <!-- PANEL ADMINA -->
    <div id="adminPanel" class="card hidden">
      <h3 style="margin-bottom: 12px; color: var(--admin);">+ Dodaj nowy wpis na bloga</h3>
      <input type="text" id="postTitle" placeholder="Tytuł posta...">
      <textarea id="postContent" rows="4" placeholder="Napisz co słychać..."></textarea>
      <input type="text" id="postNote" placeholder="Opcjonalny dopisek (np. Lokalizacja / Status)...">
      <label style="display:block; font-size: 0.85rem; color: var(--muted); margin-bottom: 6px;">Wybierz zdjęcie z telefonu:</label>
      <input type="file" id="postImage" accept="image/*">
      <button id="publishPostBtn">Opublikuj post</button>
    </div>

    <!-- POSTY Z FIRESTORE -->
    <div id="postsFeed">
      <div class="card" style="text-align:center; color:var(--muted);">Ładowanie postów z Firestore...</div>
    </div>

  </div>

  <script type="module">
    import { initializeApp } from "https://www.gstatic.com/firebasejs/10.14.1/firebase-app.js";
    import {
      getFirestore,
      collection,
      addDoc,
      doc,
      updateDoc,
      deleteDoc,
      getDocs,
      query,
      where,
      onSnapshot,
      orderBy,
      arrayUnion,
      arrayRemove
    } from "https://www.gstatic.com/firebasejs/10.14.1/firebase-firestore.js";

    // WKLEJ SWÓJ FIREBASECONFIG TUTAJ:
    const firebaseConfig = {
      apiKey: "TWOJ_API_KEY",
      authDomain: "TWOJ_PROJEKT.firebaseapp.com",
      projectId: "TWOJ_PROJEKT_ID",
      storageBucket: "TWOJ_PROJEKT.appspot.com",
      messagingSenderId: "TWOJ_SENDER_ID",
      appId: "TWOJ_APP_ID"
    };

    const app = initializeApp(firebaseConfig);
    const db = getFirestore(app);

    const ADMIN_LOGIN = "Waldek8456";
    const ADMIN_PASS = "ansk1930";

    // Trzymamy tylko aktualnie zalogowaną sesję na urządzeniu
    let currentUser = JSON.parse(sessionStorage.getItem("blog_session_user")) || null;
    let currentPosts = [];

    function setSessionUser(user) {
      currentUser = user;
      if (user) {
        sessionStorage.setItem("blog_session_user", JSON.stringify(user));
      } else {
        sessionStorage.removeItem("blog_session_user");
      }
      updateUI();
    }

    function toggleAuthModal() {
      document.getElementById("authBox").classList.toggle("hidden");
    }

    function switchAuthTab(tab) {
      document.getElementById("loginForm").classList.toggle("hidden", tab !== "login");
      document.getElementById("registerForm").classList.toggle("hidden", tab !== "register");
    }

    // REJESTRACJA W KOLEKCJI "users" W FIRESTORE
    async function handleRegister() {
      const username = document.getElementById("regUser").value.trim();
      const password = document.getElementById("regPass").value.trim();

      if (!username || !password) return alert("Wpisz login i hasło!");
      if (username.toLowerCase() === ADMIN_LOGIN.toLowerCase()) return alert("Ten login jest zajęty!");

      const q = query(collection(db, "users"), where("usernameLower", "==", username.toLowerCase()));
      const snap = await getDocs(q);
      if (!snap.empty) return alert("Taki użytkownik już istnieje!");

      await addDoc(collection(db, "users"), {
        username,
        usernameLower: username.toLowerCase(),
        password,
        createdAt: new Date()
      });

      setSessionUser({ username, isAdmin: false });
      document.getElementById("regUser").value = "";
      document.getElementById("regPass").value = "";
      toggleAuthModal();
    }

    // LOGOWANIE (ADMIN LUB KONTO Z FIRESTORE)
    async function handleLogin() {
      const username = document.getElementById("loginUser").value.trim();
      const password = document.getElementById("loginPass").value.trim();

      if (username === ADMIN_LOGIN && password === ADMIN_PASS) {
        setSessionUser({ username: ADMIN_LOGIN, isAdmin: true });
        document.getElementById("loginUser").value = "";
        document.getElementById("loginPass").value = "";
        toggleAuthModal();
        return;
      }

      const q = query(
        collection(db, "users"),
        where("username", "==", username),
        where("password", "==", password)
      );
      const snap = await getDocs(q);

      if (!snap.empty) {
        const docData = snap.docs[0].data();
        setSessionUser({ username: docData.username, isAdmin: false });
        document.getElementById("loginUser").value = "";
        document.getElementById("loginPass").value = "";
        toggleAuthModal();
      } else {
        alert("Błędny login lub hasło!");
      }
    }

    function logout() {
      setSessionUser(null);
    }

    // KOMPRESJA ZDJĘCIA POD LIMIT DOKUMENTU FIRESTORE
    function compressImage(file) {
      return new Promise((resolve) => {
        const reader = new FileReader();
        reader.readAsDataURL(file);
        reader.onload = (event) => {
          const img = new Image();
          img.src = event.target.result;
          img.onload = () => {
            const canvas = document.createElement("canvas");
            const maxWidth = 500;
            const scale = maxWidth / img.width;
            canvas.width = img.width > maxWidth ? maxWidth : img.width;
            canvas.height = img.width > maxWidth ? img.height * scale : img.height;

            const ctx = canvas.getContext("2d");
            ctx.drawImage(img, 0, 0, canvas.width, canvas.height);
            resolve(canvas.toDataURL("image/jpeg", 0.6));
          };
        };
      });
    }

    // DODAWANIE POSTA DO FIRESTORE (addDoc)
    async function createPost() {
      if (!currentUser || !currentUser.isAdmin) return;

      const title = document.getElementById("postTitle").value.trim();
      const content = document.getElementById("postContent").value.trim();
      const note = document.getElementById("postNote").value.trim();
      const fileInput = document.getElementById("postImage");

      if (!title && !content && !fileInput.files.length) {
        return alert("Dodaj chociaż tytuł, treść lub zdjęcie!");
      }

      let imageData = null;
      if (fileInput.files && fileInput.files[0]) {
        imageData = await compressImage(fileInput.files[0]);
        if (imageData.length > 750000) {
          return alert("Zdjęcie po kompresji jest za duże dla dokumentu Firestore. Wybierz mniejsze.");
        }
      }

      await addDoc(collection(db, "posts"), {
        author: ADMIN_LOGIN,
        dateLabel: new Date().toLocaleString("pl-PL", { dateStyle: "short", timeStyle: "short" }),
        createdAt: new Date(),
        title,
        content,
        note,
        image: imageData,
        likes: [],
        comments: []
      });

      document.getElementById("postTitle").value = "";
      document.getElementById("postContent").value = "";
      document.getElementById("postNote").value = "";
      fileInput.value = "";
    }

    // AKTUALIZACJA DOPISKU W FIRESTORE
    async function updateNote(postId) {
      if (!currentUser || !currentUser.isAdmin) return;
      const input = document.getElementById(`note-input-${postId}`);
      const postRef = doc(db, "posts", postId);
      await updateDoc(postRef, { note: input.value.trim() });
    }

    // USUWANIE POSTA Z FIRESTORE
    async function deletePost(postId) {
      if (!currentUser || !currentUser.isAdmin) return;
      if (confirm("Usunąć ten post?")) {
        await deleteDoc(doc(db, "posts", postId));
      }
    }

    // LAJKOWANIE W FIRESTORE
    async function toggleLike(postId) {
      if (!currentUser) return alert("Zaloguj się, żeby polubić post!");
      const post = currentPosts.find((p) => p.id === postId);
      if (!post) return;

      const postRef = doc(db, "posts", postId);
      const alreadyLiked = post.likes && post.likes.includes(currentUser.username);

      await updateDoc(postRef, {
        likes: alreadyLiked ? arrayRemove(currentUser.username) : arrayUnion(currentUser.username)
      });
    }

    // DODAWANIE KOMENTARZA W FIRESTORE
    async function addComment(postId) {
      if (!currentUser) return alert("Zaloguj się, żeby dodać komentarz!");
      const input = document.getElementById(`comment-input-${postId}`);
      const text = input.value.trim();
      if (!text) return;

      const postRef = doc(db, "posts", postId);
      await updateDoc(postRef, {
        comments: arrayUnion({
          author: currentUser.username,
          isAdmin: currentUser.isAdmin,
          text,
          date: new Date().toLocaleTimeString("pl-PL", { hour: "2-digit", minute: "2-digit" })
        })
      });
      input.value = "";
    }

    // NASŁUCH NA ŻYWO Z FIRESTORE
    function subscribeToPosts() {
      const q = query(collection(db, "posts"), orderBy("createdAt", "desc"));
      onSnapshot(q, (snapshot) => {
        currentPosts = snapshot.docs.map((d) => ({ id: d.id, ...d.data() }));
        renderPosts();
      }, (err) => {
        console.error(err);
        document.getElementById("postsFeed").innerHTML =
          `<div class="card" style="color:#ff7675;">Błąd połączenia z Firestore. Sprawdź firebaseConfig i reguły bazy.</div>`;
      });
    }

    function updateUI() {
      const userBar = document.getElementById("userBar");
      const adminPanel = document.getElementById("adminPanel");
      const headerBadge = document.getElementById("headerBadge");

      if (currentUser) {
        userBar.innerHTML = `
          <span style="font-size:0.85rem; margin-right:8px;">Cześć, <b>${escapeHTML(currentUser.username)}</b></span>
          <button class="small secondary" id="logoutBtn">Wyloguj</button>
        `;
        document.getElementById("logoutBtn").addEventListener("click", logout);
        adminPanel.classList.toggle("hidden", !currentUser.isAdmin);
        headerBadge.classList.toggle("hidden", !currentUser.isAdmin);
      } else {
        userBar.innerHTML = `<button class="small" id="openAuthBtnDynamic">Zaloguj / Rejestracja</button>`;
        document.getElementById("openAuthBtnDynamic").addEventListener("click", toggleAuthModal);
        adminPanel.classList.add("hidden");
        headerBadge.classList.add("hidden");
      }
      renderPosts();
    }

    function renderPosts() {
      const feed = document.getElementById("postsFeed");
      if (currentPosts.length === 0) {
        feed.innerHTML = `<div class="card" style="text-align:center; color:var(--muted);">Brak postów w Firestore.</div>`;
        return;
      }

      feed.innerHTML = currentPosts.map((post) => {
        const likes = post.likes || [];
        const comments = post.comments || [];
        const isLiked = currentUser && likes.includes(currentUser.username);
        const isAdmin = currentUser && currentUser.isAdmin;

        return `
          <div class="card">
            <div class="post-header">
              <div>
                <span class="post-author">@${escapeHTML(post.author || ADMIN_LOGIN)}</span>
                <span class="badge-admin">AUTOR</span>
              </div>
              <span class="post-date">${escapeHTML(post.dateLabel || "")}</span>
            </div>

            ${post.title ? `<div class="post-title">${escapeHTML(post.title)}</div>` : ""}
            ${post.content ? `<div class="post-content">${escapeHTML(post.content)}</div>` : ""}
            ${post.image ? `<img src="${post.image}" class="post-img" alt="Zdjęcie">` : ""}

            ${post.note ? `
              <div class="post-note">
                <strong>Dopisek autora:</strong>
                ${escapeHTML(post.note)}
              </div>
            ` : ""}

            ${isAdmin ? `
              <div class="admin-note-box">
                <input type="text" id="note-input-${post.id}" value="${escapeHTML(post.note || "")}" placeholder="Dodaj lub zmień dopisek...">
                <button class="small secondary" data-action="note" data-id="${post.id}">Zapisz dopisek</button>
              </div>
            ` : ""}

            <div class="post-actions">
              <button class="small like-btn ${isLiked ? "liked" : ""}" data-action="like" data-id="${post.id}">
                ❤️ ${likes.length}
              </button>
              <span style="font-size:0.85rem; color:var(--muted);">💬 ${comments.length} komentarzy</span>
              ${isAdmin ? `<button class="danger" style="margin-left:auto;" data-action="delete" data-id="${post.id}">Usuń post</button>` : ""}
            </div>

            <div class="comments-section">
              ${comments.map((c) => `
                <div class="comment">
                  <div class="comment-head">
                    <span class="comment-author" style="${c.isAdmin ? "color:var(--admin);" : ""}">
                      ${escapeHTML(c.author)} ${c.isAdmin ? "(AUTOR)" : ""}
                    </span>
                    <span>${escapeHTML(c.date || "")}</span>
                  </div>
                  <div>${escapeHTML(c.text)}</div>
                </div>
              `).join("")}

              ${currentUser ? `
                <div class="comment-form">
                  <input type="text" id="comment-input-${post.id}" placeholder="Napisz komentarz...">
                  <button class="small" data-action="comment" data-id="${post.id}">Wyślij</button>
                </div>
              ` : `<div style="font-size:0.8rem; color:var(--muted); margin-top:6px;">Zaloguj się, aby komentować.</div>`}
            </div>
          </div>
        `;
      }).join("");
    }

    function escapeHTML(str) {
      return String(str).replace(/[&<>'"]/g, (tag) => ({
        "&": "&amp;", "<": "&lt;", ">": "&gt;", "'": "&#39;", '"': "&quot;"
      }[tag]));
    }

    // Podpięcie zdarzeń
    document.getElementById("openAuthBtn").addEventListener("click", toggleAuthModal);
    document.getElementById("closeAuthBtn").addEventListener("click", toggleAuthModal);
    document.getElementById("tabLoginBtn").addEventListener("click", () => switchAuthTab("login"));
    document.getElementById("tabRegisterBtn").addEventListener("click", () => switchAuthTab("register"));
    document.getElementById("loginSubmitBtn").addEventListener("click", handleLogin);
    document.getElementById("registerSubmitBtn").addEventListener("click", handleRegister);
    document.getElementById("publishPostBtn").addEventListener("click", createPost);

    document.getElementById("postsFeed").addEventListener("click", async (e) => {
      const btn = e.target.closest("button[data-action]");
      if (!btn) return;
      const action = btn.dataset.action;
      const postId = btn.dataset.id;

      if (action === "like") await toggleLike(postId);
      else if (action === "comment") await addComment(postId);
      else if (action === "note") await updateNote(postId);
      else if (action === "delete") await deletePost(postId);
    });

    updateUI();
    subscribeToPosts();
  </script>
</body>
</html>
