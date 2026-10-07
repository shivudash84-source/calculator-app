const expressionEl = document.getElementById('expression');
const resultEl = document.getElementById('result');

let expression = '';
let lastResult = '0';

function updateDisplay() {
  expressionEl.textContent = expression || '0';
  resultEl.textContent = lastResult;
}

function appendValue(value) {
  if (value === '%') {
    expression += '/100';
    return updateDisplay();
  }

  if (value === '^') {
    expression += '^';
    return updateDisplay();
  }

  if (value === 'x²') {
    expression += '^2';
    return updateDisplay();
  }

  if (value === 'x³') {
    expression += '^3';
    return updateDisplay();
  }

  expression += value;
  updateDisplay();
}

function clearAll() {
  expression = '';
  lastResult = '0';
  updateDisplay();
}

function deleteLast() {
  expression = expression.slice(0, -1);
  updateDisplay();
}

function sanitizeExpression(raw) {
  let sanitized = raw
    .replace(/×/g, '*')
    .replace(/÷/g, '/')
    .replace(/π/g, 'PI')
    .replace(/E|e/g, 'E')
    .replace(/\^/g, '**');

  sanitized = sanitized.replace(/sin\(/g, 'sin(');
  sanitized = sanitized.replace(/cos\(/g, 'cos(');
  sanitized = sanitized.replace(/tan\(/g, 'tan(');
  sanitized = sanitized.replace(/log\(/g, 'log(');
  sanitized = sanitized.replace(/ln\(/g, 'ln(');
  sanitized = sanitized.replace(/sqrt\(/g, 'sqrt(');

  return sanitized;
}

function powerToFunction(raw) {
  return raw.replace(/(\d+(?:\.\d+)?)\^(\d+(?:\.\d+)?)/g, 'pow($1,$2)');
}

function evaluateExpression() {
  if (!expression) {
    lastResult = '0';
    updateDisplay();
    return;
  }

  try {
    const prepared = sanitizeExpression(expression);
    const withPower = powerToFunction(prepared);

    const fn = new Function(
      'sin',
      'cos',
      'tan',
      'log',
      'ln',
      'sqrt',
      'pow',
      'PI',
      'E',
      `return ${withPower};`
    );

    const result = fn(
      (x) => Math.sin((x * Math.PI) / 180),
      (x) => Math.cos((x * Math.PI) / 180),
      (x) => Math.tan((x * Math.PI) / 180),
      (x) => Math.log10(x),
      (x) => Math.log(x),
      (x) => Math.sqrt(x),
      (base, exp) => Math.pow(base, exp),
      Math.PI,
      Math.E
    );

    if (!Number.isFinite(result)) {
      throw new Error('Invalid result');
    }

    lastResult = Number(result.toFixed(10)).toString();
    expression = lastResult;
    updateDisplay();
  } catch (error) {
    lastResult = 'Error';
    updateDisplay();
  }
}

document.querySelectorAll('.btn').forEach((button) => {
  button.addEventListener('click', () => {
    const action = button.dataset.action;
    const value = button.dataset.value;

    if (action === 'clear') {
      clearAll();
      return;
    }

    if (action === 'delete') {
      deleteLast();
      return;
    }

    if (action === 'equals') {
      evaluateExpression();
      return;
    }

    if (action === 'parenthesis-open') {
      appendValue('(');
      return;
    }

    if (action === 'parenthesis-close') {
      appendValue(')');
      return;
    }

    if (value) {
      appendValue(value);
    }
  });
});

updateDisplay();

window.addEventListener('keydown', (event) => {
  const { key } = event;

  if (/^[0-9+\-*/().]$/.test(key)) {
    appendValue(key);
  }

  if (key === 'Enter' || key === '=') {
    evaluateExpression();
  }

  if (key === 'Backspace') {
    deleteLast();
  }

  if (key.toLowerCase() === 'c') {
    clearAll();
  }
});

