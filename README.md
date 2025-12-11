# GENERADOR-DE-PAPELETAS
SWEATERS LILIANA

<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Generador de Papeletas - Cirineo</title>
    
    <link rel="icon" href="https://raw.githubusercontent.com/jesusvn1993mplvt3-ai/CIRINEO/refs/heads/main/icon.ico">
    
    <script src="https://cdn.tailwindcss.com"></script>
    <script src="https://unpkg.com/react@18/umd/react.development.js"></script>
    <script src="https://unpkg.com/react-dom@18/umd/react-dom.development.js"></script>
    <script src="https://unpkg.com/@babel/standalone/babel.min.js"></script>
    <script src="https://unpkg.com/lucide@latest"></script>
    <script src="https://cdn.jsdelivr.net/npm/jsbarcode@3.11.0/dist/JsBarcode.all.min.js"></script>

    <script src="https://www.gstatic.com/firebasejs/10.7.1/firebase-app-compat.js"></script>
    <script src="https://www.gstatic.com/firebasejs/10.7.1/firebase-firestore-compat.js"></script>

    <style>
        @import url('https://fonts.googleapis.com/css2?family=Roboto:wght@400;700&display=swap');
        
        body { font-family: 'Roboto', sans-serif; }
        
        /* Estilos de impresión */
        @media print {
            body { 
                background: none; 
                margin: 0; 
                padding: 0;
            }
            .no-print { display: none !important; }
            .print-container { 
                padding: 0 !important; 
                margin: 0 !important; 
                box-shadow: none !important;
                background: none !important;
                overflow: visible !important;
            }
            .label-wrapper {
                page-break-after: always;
                box-shadow: none !important;
                margin-bottom: 0 !important;
                padding: 0 !important;
            }
            .label-wrapper:last-child {
                page-break-after: avoid;
            }
            .papeleta {
                border: 1px solid #000;
                margin: 0;
            }
        }
    </style>

</head>
<body class="bg-slate-50">
    <div id="root"></div>

    <script type="text/babel">
        // Acceder a las funciones de React
        const { useState, useEffect, useCallback, useMemo, useRef } = React;

        // --- Configuración de Firebase ---
        const firebaseConfig = {
            apiKey: "TU_API_KEY", // Reemplaza con tu API Key
            authDomain: "TU_AUTH_DOMAIN", // Reemplaza con tu dominio
            projectId: "TU_PROJECT_ID", // Reemplaza con tu Project ID
        };
        
        // Inicializar Firebase
        if (!firebase.apps.length) {
            firebase.initializeApp(firebaseConfig);
        }
        const db = firebase.firestore();

        // --- Componentes de Interfaz y Lógica ---

        // Iconos de Lucide
        const Icon = ({ name, size = 24, className = "" }) => {
            const [iconHtml, setIconHtml] = useState(null);
            
            useEffect(() => {
                const iconFn = lucide.icons[name];
                if (iconFn) {
                    const attributes = {
                        width: size,
                        height: size,
                        class: className,
                        stroke: "currentColor",
                        strokeWidth: "2",
                        strokeLinecap: "round",
                        strokeLinejoin: "round",
                    };
                    setIconHtml(iconFn(attributes));
                }
            }, [name, size, className]);

            if (!iconHtml) return null;
            return <div dangerouslySetInnerHTML={{ __html: iconHtml }} />;
        };

        // Componente de Etiqueta de Producción (Papeleta)
        const ProductionLabel = ({ labelData }) => {
            const svgRef = useRef(null);

            const order = labelData;
            const pkgInfo = order.pkgInfo || {};
            
            // ************ CORRECCIÓN CLAVE ************
            // El Santoni.html espera el formato ORDER_ID-PACKAGE_ID
            // Aseguramos que la orden tenga su ID (order.id) y el ID de paquete (order.pkgId)
            const barcodeValue = `${order.id}-${order.pkgId}`;
            // *******************************************

            const formatTimestamp = (timestamp) => {
                if (!timestamp) return 'N/A';
                // Asumiendo que el timestamp es un objeto de Firebase Firestore
                const date = timestamp.toDate ? timestamp.toDate() : new Date(timestamp);
                return date.toLocaleDateString('es-MX', { 
                    day: '2-digit', 
                    month: '2-digit', 
                    year: 'numeric' 
                });
            };

            useEffect(() => {
                if (svgRef.current && barcodeValue) {
                    try {
                        JsBarcode(svgRef.current, barcodeValue, {
                            format: "CODE128",
                            lineColor: "#000",
                            width: 2,
                            height: 40,
                            displayValue: false, // El valor legible puede ir aparte
                            margin: 0
                        });
                    } catch (e) {
                        console.error("Error al generar código de barras", e);
                    }
                }
            }, [barcodeValue]);
            
            const totalCajas = pkgInfo.cajas && pkgInfo.cajas > 0 ? pkgInfo.cajas : 1;

            return (
                <div className="papeleta w-[200mm] h-[100mm] flex border border-black text-xs">
                    
                    {/* Sección 1: Datos de la Orden (70%) */}
                    <div className="w-[70%] border-r border-black p-2 flex flex-col justify-between">
                        
                        {/* Cabecera */}
                        <div>
                            <div className="flex justify-between items-center border-b border-black pb-1 mb-1">
                                <h1 className="text-xl font-bold">PAPELETA DE PRODUCCIÓN</h1>
                                <span className="text-sm font-bold bg-black text-white px-2 py-0.5 rounded">ID: {order.id}</span>
                            </div>
                            
                            <div className="grid grid-cols-2 gap-x-4 gap-y-1 mb-2">
                                <div className="font-bold">Cliente: <span className="font-normal">{order.cliente}</span></div>
                                <div className="font-bold">No. Pedido: <span className="font-normal">{order.noPedido}</span></div>
                                <div className="font-bold">Modelo: <span className="font-normal">{order.modelo}</span></div>
                                <div className="font-bold">Color: <span className="font-normal">{order.color}</span></div>
                                <div className="font-bold">Fecha Pedido: <span className="font-normal">{formatTimestamp(order.fechaPedido)}</span></div>
                                <div className="font-bold">Fecha Entrega: <span className="font-normal">{formatTimestamp(order.fechaEntrega)}</span></div>
                            </div>
                        </div>

                        {/* Piezas del Paquete */}
                        <div className="flex-grow pt-2">
                            <h2 className="text-sm font-bold border-t border-black pt-1 mb-1">DETALLE DEL PAQUETE ({order.paqueteTipo})</h2>
                            <div className="grid grid-cols-5 gap-1 text-center font-bold text-sm border-b border-black">
                                <div className="col-span-2">TALLA</div>
                                <div className="col-span-3">CANTIDAD (Pzs)</div>
                            </div>
                            
                            <div className="max-h-[35mm] overflow-y-hidden">
                                {order.tallas && Object.entries(order.tallas).map(([talla, cantidad]) => (
                                    <div key={talla} className="grid grid-cols-5 gap-1 text-center py-0.5">
                                        <div className="col-span-2">{talla}</div>
                                        <div className="col-span-3 font-bold text-base">{cantidad}</div>
                                    </div>
                                ))}
                            </div>
                        </div>

                        {/* Firma y Observaciones */}
                        <div className="border-t border-black pt-1 mt-2">
                            <div className="flex justify-between text-xs">
                                <div className="w-1/2 pr-2">
                                    <p className="font-bold">OBSERVACIONES:</p>
                                    <p className="border-b border-dashed border-black h-4"></p>
                                </div>
                                <div className="w-1/2 pl-2">
                                    <p className="font-bold">TEJEDOR (Nombre y Firma):</p>
                                    <p className="border-b border-dashed border-black h-4"></p>
                                </div>
                            </div>
                        </div>
                    </div>
                    
                    {/* Sección 2: Barcode y Resumen (30%) */}
                    <div className="w-[30%] p-2 flex flex-col justify-center items-center">
                        <div className="text-center mb-4">
                            <div className="text-3xl font-bold mb-1">Paquete: <span className="text-red-600">{pkgInfo.codigoUnico}</span></div>
                            <div className="text-xl font-bold">Caja <span className="text-red-600">{order.cajaIndex}</span> de <span className="text-red-600">{totalCajas}</span></div>
                        </div>

                        {/* Código de Barras */}
                        <div className="flex flex-col items-center">
                            <svg ref={svgRef} className="w-full"></svg>
                            <span className="text-xs mt-1 font-mono">{barcodeValue}</span>
                        </div>
                        
                        {/* QR Code o Espacio extra */}
                        <div className="mt-4 text-center">
                            <p className="font-bold text-sm">TOTAL PZS EN ESTE PAQUETE:</p>
                            <p className="text-4xl font-extrabold text-blue-600">{order.totalPzsPaquete}</p>
                        </div>
                    </div>

                </div>
            );
        };
        
        // Función principal para generar los datos de la papeleta
        const generatePackageLabels = (selectedOrder) => {
            if (!selectedOrder || !selectedOrder.progreso) return [];
            
            const labels = [];
            
            // Si el progreso es un array, se genera una etiqueta por cada paquete/progreso
            selectedOrder.progreso.forEach((pkg, index) => {
                // Combina los datos de la orden con los datos específicos del paquete
                labels.push({
                    ...selectedOrder,
                    pkgId: pkg.id, // ID ÚNICO DEL PAQUETE
                    pkgInfo: pkg,
                    paqueteTipo: pkg.tipo || 'Mixto',
                    tallas: pkg.tallas,
                    totalPzsPaquete: pkg.totalPzs,
                    cajaIndex: index + 1 // Asume que el progreso está ordenado por caja
                });
            });

            return labels;
        };
        
        // Componente Principal de la Aplicación
        const App = () => {
            const [orders, setOrders] = useState([]);
            const [selectedOrderId, setSelectedOrderId] = useState(null);
            const [loading, setLoading] = useState(true);

            // Fetch de Órdenes desde Firebase
            useEffect(() => {
                const unsubscribe = db.collection('orders')
                    .where('status', 'in', ['PENDIENTE', 'EN PROCESO'])
                    .orderBy('fechaPedido', 'desc')
                    .onSnapshot(snapshot => {
                        const fetchedOrders = snapshot.docs.map(doc => ({
                            id: doc.id,
                            ...doc.data()
                        }));
                        setOrders(fetchedOrders);
                        setLoading(false);
                    }, error => {
                        console.error("Error al obtener órdenes:", error);
                        setLoading(false);
                    });

                return () => unsubscribe();
            }, []);

            // Orden seleccionada y generación de etiquetas
            const selectedOrder = useMemo(() => orders.find(o => o.id === selectedOrderId), [orders, selectedOrderId]);
            const packageLabels = useMemo(() => generatePackageLabels(selectedOrder), [selectedOrder]);

            const handlePrint = () => {
                window.print();
            };

            return (
                <div className="flex h-screen overflow-hidden">
                    
                    {/* Columna de Órdenes (Sidebar) */}
                    <div className="no-print w-80 bg-slate-800 text-white flex flex-col">
                        <div className="p-4 border-b border-slate-700">
                            <h2 className="text-xl font-bold flex items-center">
                                <Icon name="printer" size={20} className="mr-2" />
                                Generador de Papeletas
                            </h2>
                        </div>
                        
                        {/* Lista de Órdenes */}
                        <div className="flex-grow overflow-y-auto p-3">
                            {loading ? (
                                <div className="text-center py-10 text-slate-400">Cargando órdenes...</div>
                            ) : orders.length > 0 ? (
                                orders.map(order => (
                                    <div 
                                        key={order.id}
                                        className={`p-3 mb-2 rounded cursor-pointer transition-colors ${selectedOrderId === order.id ? 'bg-indigo-600 shadow-md' : 'bg-slate-700 hover:bg-slate-600'}`}
                                        onClick={() => setSelectedOrderId(order.id)}
                                    >
                                        <div className="font-bold text-sm">{order.cliente} - {order.modelo}</div>
                                        <div className="text-xs text-slate-300">Pedido: {order.noPedido} | ID: {order.id}</div>
                                        <div className="text-xs text-yellow-400 mt-1">{order.progreso ? order.progreso.length : 0} Paquetes</div>
                                    </div>
                                ))
                            ) : (
                                <div className="text-center py-10 text-slate-400">No hay órdenes pendientes.</div>
                            )}
                        </div>
                    </div>

                    {/* Contenido Principal (Visualización de Papeletas) */}
                    <div className="flex-grow flex flex-col">
                        
                        {/* Header con Botón de Imprimir */}
                        <div className="no-print p-4 bg-white border-b border-slate-200 shadow-sm flex justify-between items-center">
                            <h1 className="text-2xl font-bold text-slate-800">Visualización de Papeletas</h1>
                            {packageLabels.length > 0 && (
                                <button
                                    onClick={handlePrint}
                                    className="bg-green-500 hover:bg-green-600 text-white font-bold py-2 px-4 rounded-lg flex items-center transition duration-200"
                                >
                                    <Icon name="printer" size={20} className="mr-2" />
                                    Imprimir {packageLabels.length} Papeleta(s)
                                </button>
                            )}
                        </div>

                        {/* Área de Visualización (devuelve las etiquetas para la impresión) */}
                        <div className="flex-grow flex flex-col items-center p-10 bg-slate-200 overflow-y-auto print-container">
                            {packageLabels.length > 0 ? (
                                packageLabels.map((labelData, index) => (
                                    // wrapper div to control margin in screen and page-break in print
                                    <div key={index} className="label-wrapper mb-8 p-4 bg-white rounded shadow-xl hover:shadow-2xl transition-shadow">
                                        <ProductionLabel labelData={labelData} />
                                    </div>
                                ))
                            ) : (
                                <div className="text-center text-slate-400 flex flex-col items-center mt-20">
                                    <Icon name="arrow-left-circle" size={48} className="mb-4 opacity-50"/>
                                    <p className="text-lg font-bold">Selecciona una orden del menú izquierdo</p>
                                    <p className="text-sm">Para generar y visualizar sus papeletas.</p>
                                </div>
                            )}
                        </div>

                    </div>
                </div>
            );
        };

        const root = ReactDOM.createRoot(document.getElementById('root'));
        root.render(<App />);
    </script>
</body>
</html>
