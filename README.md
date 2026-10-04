// Sample Mock Dataset (Preserving ID structure & Seed fields)
const LOCAL_SEED_RECORDS = [
  {
    id: "rec_001",
    title: "Junior Frontend Engineer",
    company: "PixelCraft Labs",
    domain: "Frontend Development",
    workMode: "Remote",
    location: "Bengaluru, India",
    stipend: "₹18,000 / month",
    duration: "3 Months",
    skills: ["HTML5", "CSS3", "JavaScript", "Git"]
  },
  {
    id: "rec_002",
    title: "Node.js Backend Intern",
    company: "CloudScale Systems",
    domain: "Backend Development",
    workMode: "Hybrid",
    location: "Noida, India",
    stipend: "₹22,000 / month",
    duration: "6 Months",
    skills: ["Node.js", "Express", "MongoDB", "REST APIs"]
  },
  {
    id: "rec_003",
    title: "Full Stack Web Apprentice",
    company: "InnoTech Solutions",
    domain: "Full Stack Development",
    workMode: "Remote",
    location: "Gurugram, India",
    stipend: "₹25,000 / month",
    duration: "4 Months",
    skills: ["JavaScript", "React", "Node.js", "PostgreSQL"]
  },
  {
    id: "rec_004",
    title: "Product Design (UI/UX) Intern",
    company: "DesignHatch Studio",
    domain: "UI/UX Design",
    workMode: "On-site",
    location: "Delhi, India",
    stipend: "₹15,000 / month",
    duration: "3 Months",
    skills: ["Figma", "User Research", "Wireframing", "WCAG"]
  },
  {
    id: "rec_005",
    title: "Data Analytics Intern",
    company: "MetricHive Analytics",
    domain: "Data Analytics",
    workMode: "Remote",
    location: "Mumbai, India",
    stipend: "₹20,000 / month",
    duration: "6 Months",
    skills: ["Python", "SQL", "Pandas", "PowerBI"]
  },
  {
    id: "rec_006",
    title: "React Developer Intern",
    company: "AppSphere Tech",
    domain: "Frontend Development",
    workMode: "Hybrid",
    location: "Pune, India",
    stipend: "₹20,000 / month",
    duration: "3 Months",
    skills: ["React", "JavaScript", "Tailwind CSS", "Redux"]
  }
];

// State Management
const state = {
  allRecords: [],
  filteredRecords: [],
  filters: {
    query: "",
    domain: "all",
    workMode: "all"
  },
  loading: false,
  error: null
};

// DOM References
const searchInput = document.getElementById("searchInput");
const domainFilter = document.getElementById("domainFilter");
const workModeFilter = document.getElementById("workModeFilter");
const cardsGrid = document.getElementById("cardsGrid");
const resultsCount = document.getElementById("resultsCount");
const loadingState = document.getElementById("loadingState");
const errorState = document.getElementById("errorState");
const errorMessage = document.getElementById("errorMessage");
const emptyState = document.getElementById("emptyState");
const retryBtn = document.getElementById("retryBtn");
const resetFiltersBtn = document.getElementById("resetFiltersBtn");

// Modal DOM References
const applyModal = document.getElementById("applyModal");
const closeModalBtn = document.getElementById("closeModalBtn");
const cancelBtn = document.getElementById("cancelBtn");
const applyForm = document.getElementById("applyForm");
const modalTitle = document.getElementById("modalTitle");
const internshipIdInput = document.getElementById("internshipId");
const formSuccessMessage = document.getElementById("formSuccessMessage");

// Predictable API Envelope Simulation
async function fetchInternships() {
  state.loading = true;
  state.error = null;
  updateUIState();

  try {
    // Artificial latency for realistic loading experience
    await new Promise((resolve) => setTimeout(resolve, 350));

    // Simulated API response envelope
    const envelope = {
      status: "success",
      data: LOCAL_SEED_RECORDS,
      pagination: {
        total: LOCAL_SEED_RECORDS.length,
        page: 1,
        limit: 10
      }
    };

    if (envelope.status !== "success" || !Array.isArray(envelope.data)) {
      throw new Error("Invalid envelope payload structure.");
    }

    state.allRecords = envelope.data;
    applyFilters();
  } catch (err) {
    state.error = "Failed to load internship opportunities. Please try again.";
  } finally {
    state.loading = false;
    updateUIState();
  }
}

// Filter Logic
function applyFilters() {
  const { query, domain, workMode } = state.filters;
  const q = query.toLowerCase().trim();

  state.filteredRecords = state.allRecords.filter((record) => {
    const matchesDomain = domain === "all" || record.domain === domain;
    const matchesWorkMode = workMode === "all" || record.workMode === workMode;

    const matchesQuery =
      !q ||
      record.title.toLowerCase().includes(q) ||
      record.company.toLowerCase().includes(q) ||
      record.skills.some((skill) => skill.toLowerCase().includes(q));

    return matchesDomain && matchesWorkMode && matchesQuery;
  });

  renderCards();
  updateUIState();
}

// UI State Switcher
function updateUIState() {
  loadingState.classList.toggle("hidden", !state.loading);
  errorState.classList.toggle("hidden", !state.error);
  if (state.error) errorMessage.textContent = state.error;

  const hasData = state.filteredRecords.length > 0;
  emptyState.classList.toggle("hidden", state.loading || state.error || hasData);
  cardsGrid.classList.toggle("hidden", state.loading || state.error || !hasData);

  if (state.loading) {
    resultsCount.textContent = "Loading opportunities...";
  } else if (state.error) {
    resultsCount.textContent = "Error loading data.";
  } else {
    resultsCount.textContent = `Showing ${state.filteredRecords.length} of ${state.allRecords.length} opportunities`;
  }
}

// Dynamic Card Rendering
function renderCards() {
  cardsGrid.innerHTML = "";

  state.filteredRecords.forEach((item) => {
    const article = document.createElement("article");
    article.className = "card";
    article.setAttribute("tabindex", "0");
    article.setAttribute("aria-labelledby", `title-${item.id}`);

    const tagsHtml = item.skills
      .map((s) => `<span class="tag">${escapeHTML(s)}</span>`)
      .join("");

    article.innerHTML = `
      <div>
        <div class="card-top">
          <h2 id="title-${item.id}" class="card-title">${escapeHTML(item.title)}</h2>
          <span class="mode-badge">${escapeHTML(item.workMode)}</span>
        </div>
        <p class="card-company">${escapeHTML(item.company)} &bull; ${escapeHTML(item.location)}</p>
        <div class="card-tags">${tagsHtml}</div>
        <div class="card-meta">
          <span><strong>Stipend:</strong> ${escapeHTML(item.stipend)}</span>
          <span><strong>Duration:</strong> ${escapeHTML(item.duration)}</span>
        </div>
      </div>
      <div class="card-footer">
        <button 
          class="btn btn-primary apply-trigger" 
          data-id="${item.id}" 
          data-role="${escapeHTML(item.title)}"
          aria-haspopup="dialog"
        >
          Apply Now
        </button>
      </div>
    `;

    cardsGrid.appendChild(article);
  });

  // Attach modal trigger listeners
  document.querySelectorAll(".apply-trigger").forEach((btn) => {
    btn.addEventListener("click", () => {
      openModal(btn.dataset.id, btn.dataset.role);
    });
  });
}

// Modal Handlers
function openModal(id, title) {
  internshipIdInput.value = id;
  modalTitle.textContent = `Apply: ${title}`;
  formSuccessMessage.classList.add("hidden");
  applyForm.reset();
  clearValidationErrors();

  if (typeof applyModal.showModal === "function") {
    applyModal.showModal();
  } else {
    applyModal.setAttribute("open", "");
  }
  document.getElementById("applicantName").focus();
}

function closeModal() {
  if (typeof applyModal.close === "function") {
    applyModal.close();
  } else {
    applyModal.removeAttribute("open");
  }
}

// Strict Client-side Validation
function validateForm() {
  clearValidationErrors();
  let isValid = true;

  const nameVal = document.getElementById("applicantName").value.trim();
  const emailVal = document.getElementById("applicantEmail").value.trim();
  const urlVal = document.getElementById("portfolioUrl").value.trim();

  // Name check
  if (!nameVal || nameVal.length < 2) {
    showFieldError("nameError", "Please enter your full name.");
    isValid = false;
  }

  // Email regex check
  const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
  if (!emailRegex.test(emailVal)) {
    showFieldError("emailError", "Please provide a valid email address.");
    isValid = false;
  }

  // Safe URL check (strictly http/https)
  try {
    const parsedUrl = new URL(urlVal);
    if (parsedUrl.protocol !== "http:" && parsedUrl.protocol !== "https:") {
      showFieldError("urlError", "URL must begin with http:// or https://");
      isValid = false;
    }
  } catch (_) {
    showFieldError("urlError", "Enter a complete valid URL (e.g. https://github.com/...)");
    isValid = false;
  }

  return isValid;
}

function showFieldError(id, msg) {
  document.getElementById(id).textContent = msg;
}

function clearValidationErrors() {
  document.getElementById("nameError").textContent = "";
  document.getElementById("emailError").textContent = "";
  document.getElementById("urlError").textContent = "";
}

// Security utility: Escape HTML to prevent injection
function escapeHTML(str) {
  return String(str).replace(/[&<>'"]/g, 
    tag => ({
      '&': '&amp;',
      '<': '&lt;',
      '>': '&gt;',
      "'": '&#39;',
      '"': '&quot;'
    }[tag] || tag)
  );
}

// Event Listeners
searchInput.addEventListener("input", (e) => {
  state.filters.query = e.target.value;
  applyFilters();
});

domainFilter.addEventListener("change", (e) => {
  state.filters.domain = e.target.value;
  applyFilters();
});

workModeFilter.addEventListener("change", (e) => {
  state.filters.workMode = e.target.value;
  applyFilters();
});

resetFiltersBtn.addEventListener("click", () => {
  searchInput.value = "";
  domainFilter.value = "all";
  workModeFilter.value = "all";
  state.filters = { query: "", domain: "all", workMode: "all" };
  applyFilters();
});

retryBtn.addEventListener("click", fetchInternships);
closeModalBtn.addEventListener("click", closeModal);
cancelBtn.addEventListener("click", closeModal);

// Close on Escape or click outside
applyModal.addEventListener("click", (e) => {
  const rect = applyModal.getBoundingClientRect();
  const isInDialog = (
    rect.top <= e.clientY &&
    e.clientY <= rect.top + rect.height &&
    rect.left <= e.clientX &&
    e.clientX <= rect.left + rect.width
  );
  if (!isInDialog) closeModal();
});

applyForm.addEventListener("submit", (e) => {
  e.preventDefault();
  if (!validateForm()) return;

  // Masked transmission (Zero sensitive logging)
  formSuccessMessage.classList.remove("hidden");
  applyForm.reset();

  setTimeout(() => {
    closeModal();
  }, 1200);
});

// Initial Mount
fetchInternships();