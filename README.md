<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Registro Diario de Alimentos y Peso</title>
    <!-- Tailwind CSS CDN -->
    <script src="https://cdn.tailwindcss.com"></script>
    <!-- Inter font -->
    <link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700&display=swap" rel="stylesheet">
    <style>
        body {
            font-family: 'Inter', sans-serif;
        }
    </style>
</head>
<body>
    <div id="root"></div>

    <!-- React and ReactDOM CDN -->
    <script crossorigin src="https://unpkg.com/react@18/umd/react.production.min.js"></script>
    <script crossorigin src="https://unpkg.com/react-dom@18/umd/react-dom.production.min.js"></script>
    <!-- Babel for JSX transformation and TypeScript support -->
    <script src="https://unpkg.com/@babel/standalone/babel.min.js" data-presets="react,typescript"></script>
    <!-- SheetJS (xlsx) CDN -->
    <script src="https://unpkg.com/xlsx/dist/xlsx.full.min.js"></script>
    <!-- FileSaver.js CDN (often used by xlsx.writeFile for browser downloads) -->
    <script src="https://unpkg.com/file-saver@2.0.5/dist/FileSaver.min.js"></script>

    <script type="text/babel">
        // Definiciones de tipos para las entradas de alimentos y peso
        type FoodEntry = {
            date: string;
            mealType: string;
            time: string;
            food: string;
        };

        type WeightEntry = {
            date: string;
            weight: number;
        };

        type DataEntry = FoodEntry | WeightEntry;

        const App = () => {
            // Estados para las entradas de alimentos y peso
            const [foodEntries, setFoodEntries] = React.useState([]);
            const [weightEntries, setWeightEntries] = React.useState([]);
            
            // Estados para los campos del formulario
            const [mealType, setMealType] = React.useState('Desayuno');
            const [food, setFood] = React.useState('');
            const [weight, setWeight] = React.useState(null);
            const [emailRecipient, setEmailRecipient] = React.useState('');
            
            // Estados para los datos generales del usuario
            const [userName, setUserName] = React.useState('');
            const [userAge, setUserAge] = React.useState(null);
            const [currentWeight, setCurrentWeight] = React.useState(null);

            // useEffect para cargar datos desde localStorage al iniciar la app
            React.useEffect(() => {
                const storedFoodEntries = localStorage.getItem('foodEntries');
                const storedWeightEntries = localStorage.getItem('weightEntries');
                const storedUserName = localStorage.getItem('userName');
                const storedUserAge = localStorage.getItem('userAge');
                const storedCurrentWeight = localStorage.getItem('currentWeight');

                if (storedFoodEntries) {
                    setFoodEntries(JSON.parse(storedFoodEntries));
                }
                if (storedWeightEntries) {
                    setWeightEntries(JSON.parse(storedWeightEntries));
                }
                if (storedUserName) {
                    setUserName(storedUserName);
                }
                if (storedUserAge) {
                    setUserAge(JSON.parse(storedUserAge));
                }
                if (storedCurrentWeight) {
                    setCurrentWeight(JSON.parse(storedCurrentWeight));
                }
            }, []);

            // useEffect para guardar datos en localStorage cuando cambian las entradas
            React.useEffect(() => {
                localStorage.setItem('foodEntries', JSON.stringify(foodEntries));
                localStorage.setItem('weightEntries', JSON.stringify(weightEntries));
            }, [foodEntries, weightEntries]);

            // useEffect para guardar datos generales del usuario
            React.useEffect(() => {
                localStorage.setItem('userName', userName);
                localStorage.setItem('userAge', JSON.stringify(userAge));
                localStorage.setItem('currentWeight', JSON.stringify(currentWeight));
            }, [userName, userAge, currentWeight]);

            // Manejador para enviar entradas de alimentos
            const handleFoodSubmit = (e: React.FormEvent) => {
                e.preventDefault();
                const now = new Date();
                const date = now.toLocaleDateString();
                const time = now.toLocaleTimeString();

                const newEntry: FoodEntry = {
                    date,
                    mealType,
                    time,
                    food,
                };

                setFoodEntries([...foodEntries, newEntry]);
                setFood('');
                alert('Entrada de alimento añadida.'); // Usando alert para feedback en web
            };

            // Manejador para enviar entradas de peso
            const handleWeightSubmit = (e: React.FormEvent) => {
                e.preventDefault();
                if (weight === null || isNaN(weight)) {
                    alert('Por favor, ingresa un peso válido.'); // Usando alert para feedback en web
                    return;
                }

                const now = new Date();
                const date = now.toLocaleDateString();

                const newEntry: WeightEntry = {
                    date,
                    weight,
                };

                setWeightEntries([...weightEntries, newEntry]);
                setWeight(null);
                alert('Entrada de peso añadida.'); // Usando alert para feedback en web
            };

            // Manejador para guardar datos generales
            const handleGeneralDataSubmit = (e: React.FormEvent) => {
                e.preventDefault();
                alert('Datos generales guardados.'); // Usando alert para feedback en web
            };

            // Manejador para exportar datos a Excel
            const handleExport = () => {
                const allData: DataEntry[] = [...foodEntries, ...weightEntries];

                const excelData = allData.map((item) => {
                    if ('food' in item) {
                        return {
                            Fecha: item.date,
                            'Tipo de Comida': item.mealType,
                            Hora: item.time,
                            'Alimentos Consumidos': item.food,
                            Peso: null,
                        };
                    } else {
                        return {
                            Fecha: item.date,
                            'Tipo de Comida': null,
                            Hora: null,
                            'Alimentos Consumidos': null,
                            Peso: item.weight,
                        };
                    }
                });

                const worksheet = XLSX.utils.json_to_sheet(excelData);
                const workbook = XLSX.utils.book_new();
                XLSX.utils.book_append_sheet(workbook, worksheet, 'Hoja1');
                XLSX.writeFile(workbook, 'registro_alimentos_peso.xlsx');
            };

            // Manejador para enviar datos por correo electrónico (mailto link)
            const handleEmailExport = () => {
                if (!emailRecipient) {
                    alert('Por favor, ingresa una dirección de correo electrónico.');
                    return;
                }

                let emailBody = "Registro Diario de Alimentos y Peso:\n\n";
                
                emailBody += "--- Datos Generales ---\n";
                emailBody += `Nombre: ${userName || 'No especificado'}\n`;
                emailBody += `Edad: ${userAge !== null ? userAge : 'No especificado'}\n`;
                emailBody += `Peso Actual (general): ${currentWeight !== null ? currentWeight + ' kg' : 'No especificado'}\n\n`;

                emailBody += "--- Registro de Alimentos ---\n";
                if (foodEntries.length === 0) {
                    emailBody += "No hay entradas de alimentos.\n";
                } else {
                    foodEntries.forEach(entry => {
                        emailBody += `${entry.date} - ${entry.time} - ${entry.mealType}: ${entry.food}\n`;
                    });
                }
                
                emailBody += "\n--- Registro de Peso (Diario) ---\n";
                if (weightEntries.length === 0) {
                    emailBody += "No hay entradas de peso.\n";
                } else {
                    weightEntries.forEach(entry => {
                        emailBody += `${entry.date} - Peso: ${entry.weight} kg\n`;
                    });
                }

                const subject = encodeURIComponent("Mi Registro Diario de Alimentos y Peso");
                const body = encodeURIComponent(emailBody);
                
                // Construir el enlace mailto
                const mailtoLink = `mailto:${emailRecipient}?subject=${subject}&body=${body}`;
                
                // Abrir el enlace mailto
                window.location.href = mailtoLink;
            };

            return (
                <div className="bg-gray-100 min-h-screen py-8 px-4 flex items-center justify-center">
                    <div className="max-w-3xl w-full mx-auto bg-white rounded-xl shadow-md overflow-hidden">
                        <div className="p-8">
                            <h1 className="text-3xl font-bold text-gray-800 mb-8 text-center">Registro Diario de Alimentos y Peso</h1>

                            {/* Sección de Datos Generales */}
                            <form onSubmit={handleGeneralDataSubmit} className="mb-8 p-6 border border-gray-200 rounded-lg shadow-sm">
                                <h2 className="text-xl font-semibold text-gray-700 mb-4">Datos Generales</h2>
                                <div className="mb-4">
                                    <label htmlFor="userName" className="block text-gray-700 text-sm font-bold mb-2">
                                        Nombre:
                                    </label>
                                    <input
                                        type="text"
                                        id="userName"
                                        className="shadow appearance-none border rounded w-full py-2 px-3 text-gray-700 leading-tight focus:outline-none focus:ring-2 focus:ring-blue-500 focus:border-transparent transition duration-200 ease-in-out"
                                        placeholder="ej. Juan Pérez"
                                        value={userName}
                                        onChange={(e) => setUserName(e.target.value)}
                                    />
                                </div>
                                <div className="mb-4">
                                    <label htmlFor="userAge" className="block text-gray-700 text-sm font-bold mb-2">
                                        Edad:
                                    </label>
                                    <input
                                        type="number"
                                        id="userAge"
                                        className="shadow appearance-none border rounded w-full py-2 px-3 text-gray-700 leading-tight focus:outline-none focus:ring-2 focus:ring-blue-500 focus:border-transparent transition duration-200 ease-in-out"
                                        placeholder="ej. 30"
                                        value={userAge === null ? '' : userAge}
                                        onChange={(e) => setUserAge(parseInt(e.target.value) || null)}
                                    />
                                </div>
                                <div className="mb-6">
                                    <label htmlFor="currentWeight" className="block text-gray-700 text-sm font-bold mb-2">
                                        Peso Actual (kg):
                                    </label>
                                    <input
                                        type="number"
                                        id="currentWeight"
                                        className="shadow appearance-none border rounded w-full py-2 px-3 text-gray-700 leading-tight focus:outline-none focus:ring-2 focus:ring-blue-500 focus:border-transparent transition duration-200 ease-in-out"
                                        placeholder="ej. 70.5"
                                        value={currentWeight === null ? '' : currentWeight}
                                        onChange={(e) => setCurrentWeight(parseFloat(e.target.value) || null)}
                                    />
                                </div>
                                <button className="w-full bg-indigo-600 hover:bg-indigo-700 text-white font-bold py-2 px-4 rounded-lg focus:outline-none focus:ring-2 focus:ring-indigo-500 focus:ring-opacity-50 transition duration-200 ease-in-out transform hover:scale-105" type="submit">
                                    Guardar Datos Generales
                                </button>
                            </form>

                            {/* Display General Data */}
                            <div className="mb-8 p-6 border border-gray-200 rounded-lg shadow-sm">
                                <h2 className="text-xl font-semibold text-gray-700 mb-4">Información General</h2>
                                <p className="text-gray-700 text-base mb-2">
                                    <span className="font-medium">Nombre:</span> {userName || 'No especificado'}
                                </p>
                                <p className="text-gray-700 text-base mb-2">
                                    <span className="font-medium">Edad:</span> {userAge !== null ? userAge : 'No especificado'}
                                </p>
                                <p className="text-gray-700 text-base">
                                    <span className="font-medium">Peso Actual:</span> {currentWeight !== null ? currentWeight + ' kg' : 'No especificado'}
                                </p>
                            </div>

                            {/* Food Logging Form */}
                            <form onSubmit={handleFoodSubmit} className="mb-8 p-6 border border-gray-200 rounded-lg shadow-sm">
                                <h2 className="text-xl font-semibold text-gray-700 mb-4">Registrar Alimento</h2>
                                <div className="mb-4">
                                    <label htmlFor="mealType" className="block text-gray-700 text-sm font-bold mb-2">
                                        Tipo de Comida
                                    </label>
                                    <select
                                        id="mealType"
                                        className="shadow appearance-none border rounded w-full py-2 px-3 text-gray-700 leading-tight focus:outline-none focus:ring-2 focus:ring-blue-500 focus:border-transparent transition duration-200 ease-in-out"
                                        value={mealType}
                                        onChange={(e) => setMealType(e.target.value)}
                                    >
                                        <option>Desayuno</option>
                                        <option>Colación</option>
                                        <option>Comida</option>
                                        <option>Colación</option>
                                        <option>Cena</option>
                                    </select>
                                </div>
                                <div className="mb-6">
                                    <label htmlFor="food" className="block text-gray-700 text-sm font-bold mb-2">
                                        Alimento(s) Consumido(s)
                                    </label>
                                    <input
                                        type="text"
                                        id="food"
                                        className="shadow appearance-none border rounded w-full py-2 px-3 text-gray-700 leading-tight focus:outline-none focus:ring-2 focus:ring-blue-500 focus:border-transparent transition duration-200 ease-in-out"
                                        placeholder="ej. Avena con frutos rojos, café"
                                        value={food}
                                        onChange={(e) => setFood(e.target.value)}
                                        required
                                    />
                                </div>
                                <button className="w-full bg-blue-600 hover:bg-blue-700 text-white font-bold py-2 px-4 rounded-lg focus:outline-none focus:ring-2 focus:ring-blue-500 focus:ring-opacity-50 transition duration-200 ease-in-out transform hover:scale-105" type="submit">
                                    Añadir Entrada de Alimento
                                </button>
                            </form>

                            {/* Weight Logging Form */}
                            <form onSubmit={handleWeightSubmit} className="mb-8 p-6 border border-gray-200 rounded-lg shadow-sm">
                                <h2 className="text-xl font-semibold text-gray-700 mb-4">Registrar Peso (Diario)</h2>
                                <div className="mb-6">
                                    <label htmlFor="weight" className="block text-gray-700 text-sm font-bold mb-2">
                                        Peso (en kg)
                                    </label>
                                    <input
                                        type="number"
                                        id="weight"
                                        className="shadow appearance-none border rounded w-full py-2 px-3 text-gray-700 leading-tight focus:outline-none focus:ring-2 focus:ring-blue-500 focus:border-transparent transition duration-200 ease-in-out"
                                        placeholder="ej. 68.5"
                                        value={weight === null ? '' : weight}
                                        onChange={(e) => setWeight(parseFloat(e.target.value) || null)}
                                        required
                                    />
                                </div>
                                <button className="w-full bg-blue-600 hover:bg-blue-700 text-white font-bold py-2 px-4 rounded-lg focus:outline-none focus:ring-2 focus:ring-blue-500 focus:ring-opacity-50 transition duration-200 ease-in-out transform hover:scale-105" type="submit">
                                    Añadir Entrada de Peso
                                </button>
                            </form>

                            {/* Food Log Display */}
                            <div className="mb-8 p-6 border border-gray-200 rounded-lg shadow-sm">
                                <h2 className="text-xl font-semibold text-gray-700 mb-4">Registro de Alimentos</h2>
                                {foodEntries.length === 0 ? (
                                    <p className="text-gray-500 italic">Aún no hay entradas de alimentos.</p>
                                ) : (
                                    <ul className="list-disc list-inside space-y-2">
                                        {foodEntries.map((entry, index) => (
                                            <li key={index} className="text-gray-700 text-base">
                                                <span className="font-medium">{entry.date}</span> - {entry.time} - <span className="font-medium">{entry.mealType}</span>: {entry.food}
                                            </li>
                                        ))}
                                    </ul>
                                )}
                            </div>

                            {/* Weight Log Display */}
                            <div className="mb-8 p-6 border border-gray-200 rounded-lg shadow-sm">
                                <h2 className="text-xl font-semibold text-gray-700 mb-4">Registro de Peso (Diario)</h2>
                                {weightEntries.length === 0 ? (
                                    <p className="text-gray-500 italic">Aún no hay entradas de peso.</p>
                                ) : (
                                    <ul className="list-disc list-inside space-y-2">
                                        {weightEntries.map((entry, index) => (
                                            <li key={index} className="text-gray-700 text-base">
                                                <span className="font-medium">{entry.date}</span> - Peso: <span className="font-medium">{entry.weight} kg</span>
                                            </li>
                                        ))}
                                    </ul>
                                )}
                            </div>

                            {/* Export Buttons Section */}
                            <div className="mt-4 p-6 border border-gray-200 rounded-lg shadow-sm">
                                <h2 className="text-xl font-semibold text-gray-700 mb-4">Opciones de Exportación</h2>
                                <button onClick={handleExport} className="w-full bg-green-600 hover:bg-green-700 text-white font-bold py-3 px-4 rounded-lg focus:outline-none focus:ring-2 focus:ring-green-500 focus:ring-opacity-50 transition duration-200 ease-in-out transform hover:scale-105 mb-4">
                                    Exportar Todos los Datos a Excel
                                </button>

                                <div className="mb-4">
                                    <label htmlFor="emailRecipient" className="block text-gray-700 text-sm font-bold mb-2">
                                        Enviar Registro por Correo Electrónico a:
                                    </label>
                                    <input
                                        type="email"
                                        id="emailRecipient"
                                        className="shadow appearance-none border rounded w-full py-2 px-3 text-gray-700 leading-tight focus:outline-none focus:ring-2 focus:ring-blue-500 focus:border-transparent transition duration-200 ease-in-out"
                                        placeholder="ej. tu_correo@ejemplo.com"
                                        value={emailRecipient}
                                        onChange={(e) => setEmailRecipient(e.target.value)}
                                    />
                                </div>
                                <button onClick={handleEmailExport} className="w-full bg-purple-600 hover:bg-purple-700 text-white font-bold py-3 px-4 rounded-lg focus:outline-none focus:ring-2 focus:ring-purple-500 focus:ring-opacity-50 transition duration-200 ease-in-out transform hover:scale-105">
                                    Enviar por Correo (Abre tu Cliente de Correo)
                                </button>
                            </div>
                        </div>
                    </div>
                </div>
            );
        };

        // Render the App component into the 'root' div
        ReactDOM.render(<App />, document.getElementById('root'));
    </script>
</body>
</html>
