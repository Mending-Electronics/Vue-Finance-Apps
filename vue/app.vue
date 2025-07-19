const { createApp, ref, onMounted, watch, computed } = Vue;

createApp({
    setup() {
        // Chart instance reference
        const chartInstance = ref(null);
        
        // UI State
        const showCategories = ref({
            capital: true,
            capitalInflation: true,
            savings: true,
            savingsInflation: false,
            etfs: true,
            etfsInflation: false
        });

        // Form inputs with default values
        const capitalBase = ref(1000);
        const epargne = ref(100);
        const dateDebut = ref("2025-01-01");
        const dateFin = ref("2050-12-31");
        const livretA = ref(3);
        const LDD = ref(2);
        const PEL = ref(2.5);
        const ETFSP500 = ref(8);
        const ETFCAC40 = ref(6);
        const ETFMonde = ref(7);
        const inflation = ref(2);
        const PEAImposition = ref(17.2);
        const CTOImpot = ref(12.8);
        const CTOImposition = ref(17.2);
        const PEABlocageAnnees = ref(5);
        const PEAImpotClotureAnticipee = ref(12.8);
        const PELBlocageAnnees = ref(4);
        const PELDureeMaxVersement = ref(10);
        const PELAvantIR = ref(12);
        const PELImpot = ref(17.2);
        const tauxImpotRevenu = ref(0);
        
        
        // Plafonds (modifiables)
        const livretAPlafond = ref(22950);
        const LDDPlafond = ref(12000);
        const PELPlafond = ref(61200);
        const PEAPlafond = ref(150000);

        // Calculation results
        const calculationDetails = ref({});
        const dataLivretA = ref([0]);
        const dataLDD = ref([0]);
        const dataPEL = ref([0]);
        const dataETFSP500 = ref([0]);
        const dataETFCAC40 = ref([0]);
        const dataETFMonde = ref([0]);
        const capitalData = ref([0]);
        const inflationAdjustedCapital = ref([0]);
        const years = ref(0);

        const dataPELAfterTax = ref([0]);

        // Destroy existing chart
        const destroyChart = () => {
            if (chartInstance.value) {
                chartInstance.value.destroy();
                chartInstance.value = null;
            }
        };



        const investmentData = computed(() => {
        return [
            { 
            name: 'Livret A', 
            value: dataLivretA.value && dataLivretA.value.length > 0 ? dataLivretA.value[dataLivretA.value.length - 1]?.toFixed(2) : 0,
            color: '#0d6efd'
            },
            { 
            name: 'LDD', 
            value: dataLDD.value && dataLDD.value.length > 0 ? dataLDD.value[dataLDD.value.length - 1]?.toFixed(2) : 0,
            color: '#20c997'
            },
            { 
            name: 'PEL', 
            value: dataPEL.value && dataPEL.value.length > 0 ? dataPEL.value[dataPEL.value.length - 1]?.toFixed(2) : 0,
            color: '#0dcaf0'
            },
            { 
            name: 'ETF SP500', 
            value: dataETFSP500.value && dataETFSP500.value.length > 0 ? dataETFSP500.value[dataETFSP500.value.length - 1]?.toFixed(2) : 0,
            color: '#d63384'
            },
            { 
            name: 'ETF CAC40', 
            value: dataETFCAC40.value && dataETFCAC40.value.length > 0 ? dataETFCAC40.value[dataETFCAC40.value.length - 1]?.toFixed(2) : 0,
            color: '#6f42c1'
            },
            { 
            name: 'ETF Monde', 
            value: dataETFMonde.value && dataETFMonde.value.length > 0 ? dataETFMonde.value[dataETFMonde.value.length - 1]?.toFixed(2) : 0,
            color: '#fd7e14'
            }
        ].filter(item => item !== undefined)
        .sort((a, b) => parseFloat(b.value) - parseFloat(a.value));
        });




        // Calculate compound interest with optional tax
        function calculateCompoundInterest(initial, monthly, rate, months, isETF = false, type = 'ETF') {
            let total = initial;
            const monthlyRate = rate / 100 / 12;
            let results = [];
            let totalInvested = initial;
            let totalInvestedArray = [initial];
            let ceilingReached = false;
            
            for (let i = 1; i <= months; i++) {
                // Vérifier si on a atteint le plafond pour les livrets
                if (type === 'LIVRET_A' && total >= livretAPlafond.value) {
                    if (!ceilingReached) {
                        ceilingReached = true;
                        // On ajoute les intérêts jusqu'à la fin de l'année en cours
                        const monthsLeftInYear = 12 - ((i - 1) % 12);
                        for (let j = 0; j < monthsLeftInYear && i + j <= months; j++) {
                            total = total * (1 + monthlyRate);
                            if ((i + j) % 12 === 0 || i + j === months) {
                                results.push(total);
                            }
                        }
                        i += monthsLeftInYear - 1;
                        continue;
                    } else {
                        // On continue à appliquer les intérêts mais sans ajouter de mensualité
                        total = total * (1 + monthlyRate);
                        if (i % 12 === 0 || i === months) {
                            results.push(total);
                        }
                        continue;
                    }
                } 
                // Même logique pour le LDD
                else if (type === 'LDD' && total >= LDDPlafond.value) {
                    if (!ceilingReached) {
                        ceilingReached = true;
                        const monthsLeftInYear = 12 - ((i - 1) % 12);
                        for (let j = 0; j < monthsLeftInYear && i + j <= months; j++) {
                            total = total * (1 + monthlyRate);
                            if ((i + j) % 12 === 0 || i + j === months) {
                                results.push(total);
                            }
                        }
                        i += monthsLeftInYear - 1;
                        continue;
                    } else {
                        total = total * (1 + monthlyRate);
                        if (i % 12 === 0 || i === months) {
                            results.push(total);
                        }
                        continue;
                    }
                } 
                // Même logique pour le PEL
                else if (type === 'PEL' && total >= PELPlafond.value) {
                    if (!ceilingReached) {
                        ceilingReached = true;
                        const monthsLeftInYear = 12 - ((i - 1) % 12);
                        for (let j = 0; j < monthsLeftInYear && i + j <= months; j++) {
                            total = total * (1 + monthlyRate);
                            if ((i + j) % 12 === 0 || i + j === months) {
                                results.push(total);
                            }
                        }
                        i += monthsLeftInYear - 1;
                        continue;
                    } else {
                        total = total * (1 + monthlyRate);
                        if (i % 12 === 0 || i === months) {
                            results.push(total);
                        }
                        continue;
                    }
                }
                
                // Si on n'a pas atteint le plafond, on ajoute la mensualité
                total += monthly;
                totalInvested += monthly;
                
                // Application des intérêts
                total = total * (1 + monthlyRate);
                
                // Enregistrement des résultats annuels
                if (i % 12 === 0 || i === months) {
                    // Pour les ETFs, on applique la taxe uniquement sur les gains à la fin
                    if (isETF && i > 60) { // Après 5 ans (60 mois)
                        const profit = total - totalInvested;
                        const tax = profit * 0.172; // 17.2% de taxe sur les plus-values
                        results.push(total - tax);
                    } else {
                        results.push(total);
                    }
                }
            }
            
            return results;
        }

        // Calculate simple capital (without interest)
        function calculateSimpleCapital(initial, monthly, months) {
            let results = [];
            let total = initial;
            for (let i = 1; i <= months; i++) {
                total += monthly;
                if (i % 12 === 0 || i === months) {
                    results.push(total);
                }
            }
            return results;
        }

        // Apply inflation to a dataset
        function applyInflation(dataset, inflationRate) {
            return dataset.map((value, index) => {
                const years = (index + 1) / 12; 
                return value / Math.pow(1 + (inflationRate / 100), years);
            });
        }







        
        // Calculate PEL after tax
        function calculatePELAfterTax(pelData) {
            return pelData.map((value, index) => {
                const years = (index + 1) / 12;
                const capitalInitial = capitalBase.value + (epargne.value * 12 * Math.min(years, PELDureeMaxVersement.value));
                const interets = value - capitalInitial;
                const impot = interets * (PELImpot.value / 100);
                return value - impot;
            });
        }












        // Get ETF investment details
        function getETFDetails(finalAmount, initial, monthly, yearsValue, rate, taxRate) {
            const totalInvested = initial + (monthly * 12 * yearsValue);
            const profit = finalAmount - totalInvested;
            const tax = profit > 0 ? profit * (taxRate / 100) : 0;
            const netProfit = profit - tax;
            const netAmount = finalAmount - tax;
            const rateOfReturn = ((netAmount - totalInvested) / totalInvested) * 100;
            
            return {
                totalInvested,
                profit,
                tax,
                netProfit,
                netAmount,
                rateOfReturn
            };
        }

        // Filter data points above ceiling - Modifié pour afficher les résultats même après le plafond
        function filterDataByCeiling(data, ceiling) {
            // On retourne simplement les données sans filtrage pour afficher tous les points
            return data;
            
            // Ancien code commenté qui masquait les points après le plafond
            /*
            return data.map((value, index) => {
                if (value > ceiling) {
                    // Si au-dessus du plafond, retourne null pour masquer le point
                    return null;
                }
                return value;
            });
            */
        }

        // Get datasets based on selected categories
        const getChartDatasets = () => {
            const datasets = [];
            
            // Capital
            if (showCategories.value.capital) {
                datasets.push({
                    label: "Capital",
                    data: capitalData.value,
                    borderColor: getComputedStyle(document.body).getPropertyValue('--bs-gray'),
                    borderDash: [5, 5],
                    borderWidth: 2
                });
            }
            
            // Capital with inflation
            if (showCategories.value.capitalInflation) {
                datasets.push({
                    label: `Capital avec inflation (${inflation.value}%)`,
                    data: inflationAdjustedCapital.value,
                    borderColor: getComputedStyle(document.body).getPropertyValue('--bs-light'),
                    borderDash: [5, 5],
                    borderWidth: 2
                });
            }
            
            // Savings without inflation
            if (showCategories.value.savings) {
                datasets.push(
                    { 
                        label: `Livret A (${livretA.value}%)`, 
                        data: filterDataByCeiling(dataLivretA.value, livretAPlafond.value),
                        borderColor: getComputedStyle(document.body).getPropertyValue('--bs-blue'),
                        borderWidth: 2
                    },
                    { 
                        label: `LDD (${LDD.value}%)`, 
                        data: filterDataByCeiling(dataLDD.value, LDDPlafond.value),
                        borderColor: getComputedStyle(document.body).getPropertyValue('--bs-purple'),
                        borderWidth: 2
                    },
                    { 
                        label: `PEL (${PEL.value}%)`, 
                        data: filterDataByCeiling(dataPEL.value, PELPlafond.value),
                        borderColor: getComputedStyle(document.body).getPropertyValue('--bs-cyan'),
                        borderWidth: 2
                    }
                );
            }
            


            // Savings with inflation
            if (showCategories.value.savingsInflation) {
                const applyInflationToData = (data) => {
                    return data.map((value, index) => {
                        const years = (index + 1) / 12;
                        return value / Math.pow(1 + (inflation.value / 100), years);
                    });
                };
                
                datasets.push(
                    { 
                        label: `Livret A (${livretA.value}% avec inflation)`, 
                        data: filterDataByCeiling(applyInflationToData(dataLivretA.value), livretAPlafond.value),
                        borderColor: getComputedStyle(document.body).getPropertyValue('--bs-blue'),
                        borderDash: [2, 2],
                        borderWidth: 1
                    },
                    { 
                        label: `LDD (${LDD.value}% avec inflation)`, 
                        data: filterDataByCeiling(applyInflationToData(dataLDD.value), LDDPlafond.value),
                        borderColor: getComputedStyle(document.body).getPropertyValue('--bs-purple'),
                        borderDash: [2, 2],
                        borderWidth: 1
                    },
                    { 
                        label: `PEL (${PEL.value}% après prélèvement Sociaux et avec inflation)`, 
                        data: dataPELAfterTax.value && dataPELAfterTax.value.length > 0 
                            ? filterDataByCeiling(applyInflationToData(dataPELAfterTax.value), PELPlafond.value) 
                            : filterDataByCeiling(applyInflationToData(dataPEL.value), PELPlafond.value),
                        borderColor: getComputedStyle(document.body).getPropertyValue('--bs-cyan'),
                        borderDash: [2, 2],
                        borderWidth: 1
                    }
                );
            }
            
            // ETFs without taxes or inflation
            if (showCategories.value.etfs) {
                const getETFDataNoTax = (rate) => {
                    return calculateCompoundInterest(capitalBase.value, epargne.value, rate, years.value * 12, false);
                };
                
                datasets.push(
                    { 
                        label: `ETF SP500 (${ETFSP500.value}%)`, 
                        data: getETFDataNoTax(ETFSP500.value), 
                        borderColor: getComputedStyle(document.body).getPropertyValue('--bs-success'),
                        borderWidth: 2
                    },
                    { 
                        label: `ETF CAC40 (${ETFCAC40.value}%)`, 
                        data: getETFDataNoTax(ETFCAC40.value), 
                        borderColor: getComputedStyle(document.body).getPropertyValue('--bs-warning'),
                        borderWidth: 2
                    },
                    { 
                        label: `ETF Monde (${ETFMonde.value}%)`, 
                        data: getETFDataNoTax(ETFMonde.value), 
                        borderColor: getComputedStyle(document.body).getPropertyValue('--bs-primary'),
                        borderWidth: 2
                    }
                );
            }
            
            // ETFs with taxes and inflation
            if (showCategories.value.etfsInflation) {
                const getETFDataWithTaxInflation = (rate) => {
                    // 1. Récupérer les valeurs brutes de l'ETF
                    const etfValues = calculateCompoundInterest(capitalBase.value, epargne.value, rate, years.value * 12, false);
                   
                    
                    // 2. Récupérer les valeurs du capital investi (sans intérêts)
                    const capitalValues = calculateSimpleCapital(capitalBase.value, epargne.value, years.value * 12);

                    
                    // 3. Calculer les valeurs après impôts et inflation
                    return etfValues.map((etfValue, index) => {

                        // Pour les 5 premières années, retourner null pour masquer les points
                        if (index < 5) {
                            return null;
                        }

                        // Valeur de l'ETF
                        const total = etfValue;
                        
                        // Capital investi
                        const capitalInvesti = capitalValues[index];
                        
                        // Gain
                        const gain = total - capitalInvesti;
                        
                        // Impôt sur le gain (17.2% de prélèvement sociaux)
                        const impot = gain > 0 ? gain * (PEAImposition.value / 100) : 0;
                        
                        // Gain après impôt
                        const gainApresImpot = gain - impot;
                        
                        // Total après impôt
                        const totalApresImpot = capitalInvesti + gainApresImpot;
                        
                        // Calcul de l'inflation (annuelle cumulée)
                        const Inflation = totalApresImpot * ((inflation.value / 100)/12);
                        
                        // Total après impôt et inflation
                        const totalFinal = totalApresImpot - Inflation;
                        
                        return totalFinal;
                    });

                    
                };
                
                datasets.push(
                    { 
                        label: `ETF SP500 (${ETFSP500.value}% après impôts + inflation)`, 
                        data: getETFDataWithTaxInflation(ETFSP500.value), 
                        borderColor: getComputedStyle(document.body).getPropertyValue('--bs-success'),
                        borderDash: [2, 2],
                        borderWidth: 1
                    },
                    { 
                        label: `ETF CAC40 (${ETFCAC40.value}% après impôts + inflation)`, 
                        data: getETFDataWithTaxInflation(ETFCAC40.value), 
                        borderColor: getComputedStyle(document.body).getPropertyValue('--bs-warning'),
                        borderDash: [2, 2],
                        borderWidth: 1
                    },
                    { 
                        label: `ETF Monde (${ETFMonde.value}% après impôts + inflation)`, 
                        data: getETFDataWithTaxInflation(ETFMonde.value), 
                        borderColor: getComputedStyle(document.body).getPropertyValue('--bs-primary'),
                        borderDash: [2, 2],
                        borderWidth: 1
                    }
                );
            }
            
            return datasets;
        };



















        // Generate the chart
        const generateChart = () => {
            destroyChart();
            
            // Calculate months and years
            const monthsCalc = Math.ceil((new Date(dateFin.value) - new Date(dateDebut.value)) / (1000 * 60 * 60 * 24 * 30));
            years.value = Math.ceil(monthsCalc / 12);
            
            // Calculate base data
            dataLivretA.value = calculateCompoundInterest(capitalBase.value, epargne.value, livretA.value, years.value * 12, false, 'LIVRET_A');
            dataLDD.value = calculateCompoundInterest(capitalBase.value, epargne.value, LDD.value, years.value * 12, false, 'LDD');
            dataPEL.value = calculateCompoundInterest(capitalBase.value, epargne.value, PEL.value, years.value * 12, false, 'PEL');
            dataETFSP500.value = calculateCompoundInterest(capitalBase.value, epargne.value, ETFSP500.value, years.value * 12, true);
            dataETFCAC40.value = calculateCompoundInterest(capitalBase.value, epargne.value, ETFCAC40.value, years.value * 12, true);
            dataETFMonde.value = calculateCompoundInterest(capitalBase.value, epargne.value, ETFMonde.value, years.value * 12, true);
            
            // Calculate capital lines
            capitalData.value = calculateSimpleCapital(capitalBase.value, epargne.value, years.value * 12);
            inflationAdjustedCapital.value = applyInflation(capitalData.value, inflation.value);

            // Generate labels
            const labels = [];
            const startYear = new Date(dateDebut.value).getFullYear();
            for (let i = 0; i <= years.value; i++) {
                labels.push(startYear + i);
            }

            // Create chart
            const ctx = document.getElementById("myChart").getContext("2d");
            chartInstance.value = new Chart(ctx, {
                type: "line",
                data: {
                    labels: labels,
                    datasets: getChartDatasets()
                },
                options: {
                    responsive: true,
                    maintainAspectRatio: true,
                    interaction: {
                        mode: 'index',
                        intersect: false,
                    },
                    plugins: {
                        tooltip: {
                            callbacks: {
                                label: function(context) {
                                    let label = context.dataset.label || '';
                                    if (label) {
                                        label += ': ';
                                    }
                                    if (context.parsed.y !== null) {
                                        label += new Intl.NumberFormat('fr-FR', { 
                                            style: 'currency', 
                                            currency: 'EUR',
                                            maximumFractionDigits: 0 
                                        }).format(context.parsed.y);
                                    }
                                    return label;
                                }
                            }
                        },
                        legend: {
                            position: 'bottom',
                            labels: {
                                boxWidth: 12,
                                padding: 20
                            }
                        }
                    },
                    scales: {
                        y: {
                            beginAtZero: true,
                            ticks: {
                                callback: function(value) {
                                    return new Intl.NumberFormat('fr-FR', { 
                                        style: 'currency', 
                                        currency: 'EUR',
                                        maximumFractionDigits: 0 
                                    }).format(value);
                                }
                            }
                        }
                    }
                }
            });

            // Update calculation details
            const sp500Details = getETFDetails(
                dataETFSP500.value[dataETFSP500.value.length - 1],
                capitalBase.value,
                epargne.value,
                years.value,
                ETFSP500.value,
                PEAImposition.value
            );

            const details = {
                '--- Paramètres de simulation': '---',
                'Capital initial': `${capitalBase.value.toLocaleString('fr-FR')} €`,
                'Épargne mensuelle': `${epargne.value.toLocaleString('fr-FR')} €`,
                'Période': `Du ${new Date(dateDebut.value).toLocaleDateString('fr-FR')} au ${new Date(dateFin.value).toLocaleDateString('fr-FR')} (${years.value} ans)`,
                '--- Détails ETF SP500': '---',
                'Total investi': `${sp500Details.totalInvested.toLocaleString('fr-FR', { minimumFractionDigits: 2, maximumFractionDigits: 2 })} €`,
                'Plus-value brute': `${sp500Details.profit.toLocaleString('fr-FR', { minimumFractionDigits: 2, maximumFractionDigits: 2 })} €`
            };

            // Add tax details
            details[`Impôt (${PEAImposition.value}%)`] = 
                `${sp500Details.tax.toLocaleString('fr-FR', { minimumFractionDigits: 2, maximumFractionDigits: 2 })} €`;

            // Add remaining details
            details['Plus-value nette'] = 
                `${sp500Details.netProfit.toLocaleString('fr-FR', { minimumFractionDigits: 2, maximumFractionDigits: 2 })} €`;
            details['Taux de rendement net'] = 
                `${sp500Details.rateOfReturn.toLocaleString('fr-FR', { minimumFractionDigits: 2, maximumFractionDigits: 2 })} %`;

            calculationDetails.value = details;
        };

        // Handle form submission
        const handleSubmit = (event) => {
            event.preventDefault();

            console.log("Submit clicked");

            // Ensure all number values are properly parsed as floats
            capitalBase.value = parseFloat(capitalBase.value) || 0;
            epargne.value = parseFloat(epargne.value) || 0;
            livretA.value = parseFloat(livretA.value) || 0;
            LDD.value = parseFloat(LDD.value) || 0;
            PEL.value = parseFloat(PEL.value) || 0;
            ETFSP500.value = parseFloat(ETFSP500.value) || 0;
            ETFCAC40.value = parseFloat(ETFCAC40.value) || 0;
            ETFMonde.value = parseFloat(ETFMonde.value) || 0;
            inflation.value = parseFloat(inflation.value) || 0;
            PEAImposition.value = parseFloat(PEAImposition.value) || 0;
            CTOImpot.value = parseFloat(CTOImpot.value) || 0;
            CTOImposition.value = parseFloat(CTOImposition.value) || 0;
            
            // Generate chart
            generateChart();
            
            // Scroll to the chart
            const chartElement = document.querySelector('canvas');
            if (chartElement) {
                chartElement.scrollIntoView({ behavior: 'smooth' });
            }

            // Calculate investment data
            dataLivretA.value = calculateCompoundInterest(capitalBase.value, epargne.value, livretA.value, years.value * 12, false, 'LIVRET_A');
            dataLDD.value = calculateCompoundInterest(capitalBase.value, epargne.value, LDD.value, years.value * 12, false, 'LDD');
            dataPEL.value = calculateCompoundInterest(capitalBase.value, epargne.value, PEL.value, years.value * 12, false, 'PEL');
            dataETFSP500.value = calculateCompoundInterest(capitalBase.value, epargne.value, ETFSP500.value, years.value * 12, true, 'ETF');
            dataETFCAC40.value = calculateCompoundInterest(capitalBase.value, epargne.value, ETFCAC40.value, years.value * 12, true, 'ETF');
            dataETFMonde.value = calculateCompoundInterest(capitalBase.value, epargne.value, ETFMonde.value, years.value * 12, true, 'ETF');
            dataPEL.value = calculateCompoundInterest(capitalBase.value, epargne.value, PEL.value, years.value * 12, false, 'PEL');
            dataPELAfterTax.value = calculatePELAfterTax(dataPEL.value);



            // À la fin de la fonction, après les calculs
            console.log("Après calculs:", {
                dataLivretA: dataLivretA.value,
                dataLDD: dataLDD.value,
                dataPEL: dataPEL.value,
                dataETFSP500: dataETFSP500.value,
                dataETFCAC40: dataETFCAC40.value,
                dataETFMonde: dataETFMonde.value
            });


        };

        // Update chart when checkboxes are toggled
        watch(showCategories, () => {
            if (chartInstance.value) {
                chartInstance.value.data.datasets = getChartDatasets();
                chartInstance.value.update();
            }
        }, { deep: true });

        // Initialize chart on mount
        onMounted(() => {
            generateChart();
            return () => destroyChart();
        });

        return {
            capitalBase,
            epargne,
            dateDebut,
            dateFin,
            livretA,
            LDD,
            PEL,
            ETFSP500,
            ETFCAC40,
            ETFMonde,
            inflation,
            PEAImposition,
            showCategories,
            calculationDetails,
            generateChart,
            handleSubmit,
            PEABlocageAnnees,
            PEAImpotClotureAnticipee,
            PELBlocageAnnees,
            PELDureeMaxVersement,
            PELAvantIR,
            tauxImpotRevenu,
            livretAPlafond,
            LDDPlafond,
            PELPlafond,
            PELImpot,
            PEAPlafond,
            CTOImpot,
            CTOImposition,

            dataLivretA,
            dataLDD,
            dataPEL,
            dataETFSP500,
            dataETFCAC40,
            dataETFMonde,
            capitalData,
            inflationAdjustedCapital,
            years,
            dataPELAfterTax,

            investmentData
        };
    }
}).mount('#app');