[drawio-plugin.js](https://github.com/user-attachments/files/22711130/drawio-plugin.js)[Uploa// Draw.io Python Plugin
Draw.loadPlugin(function(ui) {
    const { actions, editor, menubar, toolbar } = ui;
    const graph = editor.graph;
    
    console.log('Python Plugin loaded successfully!');
    
    // Конфигурация
    const PYTHON_BACKEND_URL = 'http://localhost:5000/api';
    
    // Функция для проверки подключения к Python
    async function checkPythonConnection() {
        try {
            const response = await fetch(`${PYTHON_BACKEND_URL}/health`);
            const result = await response.json();
            return result.status === 'OK';
        } catch (error) {
            console.error('Python backend is not available:', error);
            return false;
        }
    }
    
    // Основная функция обработки
    async function processWithPython() {
        const isConnected = await checkPythonConnection();
        
        if (!isConnected) {
            ui.alert('Ошибка', 'Python сервер не запущен! Запустите backend.py на порту 5000.');
            return;
        }
        
        try {
            // Получаем все ячейки или выделенные
            const cells = graph.getSelectionCells().length > 0 
                ? graph.getSelectionCells() 
                : graph.getModel().getCells();
            
            // Подготавливаем данные для отправки
            const cellData = Object.values(cells).map(cell => ({
                id: cell.getId(),
                value: cell.getValue() || '',
                type: cell.isEdge() ? 'edge' : 'vertex',
                style: cell.getStyle() || '',
                geometry: cell.getGeometry() ? {
                    x: cell.getGeometry().x,
                    y: cell.getGeometry().y,
                    width: cell.getGeometry().width,
                    height: cell.getGeometry().height
                } : null
            }));
            
            ui.showDialog(new SpinnerDialog().container, 300, 120, true, true);
            
            // Отправляем данные в Python
            const response = await fetch(`${PYTHON_BACKEND_URL}/process_graph`, {
                method: 'POST',
                headers: {
                    'Content-Type': 'application/json',
                },
                body: JSON.stringify(cellData)
            });
            
            const result = await response.json();
            
            ui.hideDialog();
            
            if (result.success) {
                // Применяем изменения от Python
                applyPythonChanges(result.changes);
                ui.alert('Успех', `Обработано ${result.changes.length} элементов`);
            } else {
                ui.alert('Ошибка Python', result.error);
            }
            
        } catch (error) {
            ui.hideDialog();
            ui.alert('Ошибка', `Ошибка связи: ${error.message}`);
        }
    }
    
    // Применение изменений от Python
    function applyPythonChanges(changes) {
        graph.getModel().beginUpdate();
        try {
            changes.forEach(change => {
                const cell = graph.getModel().getCell(change.cellId);
                if (cell) {
                    if (change.newValue !== undefined) {
                        cell.setValue(change.newValue);
                    }
                    if (change.newStyle) {
                        cell.setStyle(change.newStyle);
                    }
                }
            });
        } finally {
            graph.getModel().endUpdate();
            graph.refresh();
        }
    }
    
    // Функция для математических вычислений
    async function calculateWithPython() {
        try {
            // Пример: отправляем числа для вычислений
            const numbers = [1, 2, 3, 4, 5];
            
            const response = await fetch(`${PYTHON_BACKEND_URL}/calculate`, {
                method: 'POST',
                headers: {
                    'Content-Type': 'application/json',
                },
                body: JSON.stringify({ numbers: numbers })
            });
            
            const result = await response.json();
            
            if (result.success) {
                ui.alert('Результат вычислений', 
                    `Сумма: ${result.total}\nСреднее: ${result.average}\nКоличество: ${result.count}`);
            }
            
        } catch (error) {
            ui.alert('Ошибка', `Ошибка вычислений: ${error.message}`);
        }
    }
    
    // Добавляем действия
    actions.addAction('python_process', function() {
        processWithPython();
    });
    
    actions.addAction('python_calculate', function() {
        calculateWithPython();
    });
    
    actions.addAction('python_check', async function() {
        const isConnected = await checkPythonConnection();
        if (isConnected) {
            ui.alert('Проверка связи', '✅ Python сервер работает!');
        } else {
            ui.alert('Проверка связи', '❌ Python сервер не доступен!');
        }
    });
    
    // Добавляем кнопки в тулбар
    toolbar.addSeparator();
    toolbar.addItem('Python Check', 'python_check');
    toolbar.addItem('Process with Python', 'python_process');
    toolbar.addItem('Calculate', 'python_calculate');
    
    // Добавляем в меню
    menubar.addMenu('Python Plugin', function(menu, parent) {
        menu.addItem('Проверить связь с Python', 'python_check');
        menu.addItem('Обработать диаграмму', 'python_process');
        menu.addItem('Вычисления', 'python_calculate');
    });
    
    console.log('Python Plugin initialization completed!');
});ding drawio-plugin.js…]()
