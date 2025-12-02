let currentScenario = null;
let currentStage = 0;
let isFacilitator = false;
let revealedInjects = 0;

const mainContent = document.getElementById('main-content');
const prevBtn = document.getElementById('prev-btn');
const nextBtn = document.getElementById('next-btn');
const homeBtn = document.getElementById('home-btn');
const progressContainer = document.getElementById('progress-container');
const progressBar = document.getElementById('progress-bar');
const progressLabel = document.getElementById('progress-label');

function selectMode(mode) {
  isFacilitator = (mode === 'facilitator');
  document.getElementById('mode-selection').classList.add('hidden');
  loadScenarioList();
  hideProgressBar();
}

async function loadScenarioList() {
  const res = await fetch('scenarios.json');
  const scenarioList = await res.json();

  mainContent.innerHTML = '<h2 class="text-xl font-semibold mb-2">Select a scenario:</h2>';
  scenarioList.forEach(s => {
    const btn = document.createElement('button');
    btn.className = 'block w-full text-left px-4 py-2 mb-2 bg-gray-800 border border-gray-600 rounded hover:bg-gray-700';
    btn.textContent = s.title;
    btn.onclick = () => loadScenario(s.file);
    mainContent.appendChild(btn);
  });

  hideProgressBar();
}

async function loadScenario(file) {
  const res = await fetch(`scenarios/${file}`);
  currentScenario = await res.json();
  currentStage = -1;
  showIntro();
  hideProgressBar();
}

function showIntro() {
  const s = currentScenario;
  mainContent.innerHTML = `
    <h2 class="text-2xl font-bold mb-4">${s.title}</h2>
    <p class="mb-4 text-gray-300 whitespace-pre-line">${s.narrative_intro}</p>
    <p class="mb-2 font-semibold">Participants:</p>
    <ul class="list-disc list-inside mb-4">${s.participants.map(p => `<li>${p}</li>`).join('')}</ul>
    <p class="mb-2 font-semibold">Objectives:</p>
    <p class="mb-4">${s.objectives}</p>
  `;

  prevBtn.classList.add('hidden');
  nextBtn.classList.remove('hidden');
  nextBtn.disabled = false;
  nextBtn.textContent = 'Begin Scenario';
  nextBtn.classList.remove('bg-blue-600', 'bg-purple-600');
  nextBtn.classList.add('bg-green-600');
  homeBtn.classList.remove('hidden');
  hideProgressBar();
}

function startScenario() {
  currentStage = 0;
  revealedInjects = 0;
  renderStage();
}

function renderStage() {
  const stage = currentScenario.stages[currentStage];
  const injectCount = stage.injects ? stage.injects.length : 0;
  revealedInjects = 0;

  showProgressBar();
  updateProgressBar();

  let html = `
    <div>
      <p class="text-lg font-semibold mb-2">${stage.narrative}</p>
      <div class="mb-4">
        <h3 class="font-bold">Prompts:</h3>
        <ul class="list-disc list-inside">
          ${stage.prompts.map(p => `<li>${p}</li>`).join('')}
        </ul>
      </div>
  `;

  if (isFacilitator) {
    html += `
      <div class="mb-4">
        <h3 class="font-bold">Injects:</h3>
        <ul class="list-disc list-inside text-red-400">
          ${stage.injects?.map(i => `<li>${i}</li>`).join('') || '<li>None</li>'}
        </ul>
      </div>
    `;
  } else {
    html += `
      <div class="mb-4">
        <h3 class="font-bold">Injects:</h3>
        <ul id="inject-list" class="list-disc list-inside text-red-400">
          ${stage.injects?.slice(0, revealedInjects).map(i => `<li>${i}</li>`).join('') || '<li>No injects</li>'}
        </ul>
        ${revealedInjects < injectCount
          ? `<button onclick="revealNextInject()" class="mt-2 px-4 py-2 bg-red-600 text-white rounded">Reveal Next Inject</button>`
          : ''
        }
      </div>
    `;
  }

  if (isFacilitator) {
    html += `
      <div class="mt-4 p-4 bg-purple-900 border-l-4 border-purple-500">
        <h3 class="font-bold">Facilitator Notes:</h3>
        <p class="mb-2">${stage.facilitator?.notes || '—'}</p>
        <h4 class="font-semibold">Scoring Guidance:</h4>
        <pre class="whitespace-pre-wrap">${stage.facilitator?.scoring || '—'}</pre>
      </div>
    `;
  }

  html += `</div>`;
  mainContent.innerHTML = html;

  prevBtn.classList.toggle('hidden', currentStage === 0);
  homeBtn.classList.remove('hidden');

  if (isFacilitator || injectCount === 0) {
    nextBtn.disabled = false;
  } else {
    nextBtn.disabled = revealedInjects < injectCount;
  }

  nextBtn.classList.toggle('hidden', currentStage >= currentScenario.stages.length - 1);
  nextBtn.textContent = 'Next →';
  nextBtn.classList.remove('bg-green-600');
  nextBtn.classList.add(isFacilitator ? 'bg-purple-600' : 'bg-blue-600');
}

function revealNextInject() {
  const stage = currentScenario.stages[currentStage];
  const injectCount = stage.injects ? stage.injects.length : 0;

  if (revealedInjects < injectCount) {
    revealedInjects++;
    renderStage();
  }
}

function goToNextStage() {
  if (currentStage === -1) {
    startScenario();
    return;
  }

  if (currentStage < currentScenario.stages.length - 1) {
    currentStage++;
    revealedInjects = 0;
    renderStage();
  }
}

function goToPreviousStage() {
  if (currentStage > 0) {
    currentStage--;
    revealedInjects = 0;
    renderStage();
  }
}

function goHome() {
  currentScenario = null;
  currentStage = 0;
  revealedInjects = 0;
  document.getElementById('mode-selection').classList.remove('hidden');
  mainContent.innerHTML = '';
  prevBtn.classList.add('hidden');
  nextBtn.classList.add('hidden');
  homeBtn.classList.add('hidden');
  hideProgressBar();
}

function updateProgressBar() {
  if (!currentScenario || currentStage < 0) return;
  const total = currentScenario.stages.length;
  const current = currentStage + 1;
  const percent = Math.round((current / total) * 100);
  progressBar.style.width = `${percent}%`;
  progressLabel.textContent = `Stage ${current} of ${total}`;
  progressBar.className = `h-full transition-all duration-300 ${isFacilitator ? 'bg-purple-500' : 'bg-blue-500'}`;
}

function showProgressBar() {
  progressContainer.classList.remove('hidden');
}

function hideProgressBar() {
  progressContainer.classList.add('hidden');
  progressBar.style.width = '0%';
  progressLabel.textContent = '';
}

// Expose functions for HTML onclick
window.selectMode = selectMode;
window.goToNextStage = goToNextStage;
window.goToPreviousStage = goToPreviousStage;
window.goHome = goHome;
window.revealNextInject = revealNextInject;
