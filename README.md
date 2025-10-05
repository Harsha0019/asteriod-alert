# asteriod-alert
// Function to fetch and display asteroid data
function fetchAsteroids() {
    const today = new Date().toISOString().split('T')[0]; // Today's date
    const endDate = new Date();
    endDate.setDate(endDate.getDate() + 7); // Next 7 days
    const endDateStr = endDate.toISOString().split('T')[0];

    const apiUrl = `https://api.nasa.gov/neo/rest/v1/feed?start_date=${today}&end_date=${endDateStr}&api_key=DEMO_KEY`;

    fetch(apiUrl)
        .then(response => response.json())
        .then(data => {
            const nearEarthObjects = data.near_earth_objects;
            let asteroids = [];

            // Collect asteroids from all dates
            for (let date in nearEarthObjects) {
                nearEarthObjects[date].forEach(asteroid => {
                    const closeApproach = asteroid.close_approach_data[0];
                    const sizeKm = (asteroid.estimated_diameter.meters.estimated_diameter_min / 1000).toFixed(2);
                    asteroids.push({
                        name: asteroid.name.replace(/[\(\)']/g, ''), // Clean name
                        closeApproachDate: date,
                        distanceKm: parseFloat(closeApproach.miss_distance.kilometers).toLocaleString(),
                        sizeKm: sizeKm,
                        isHazardous: asteroid.is_potentially_hazardous_asteroid ? 'Yes' : 'No'
                    });
                });
            }

            displayAsteroids(asteroids);
        })
        .catch(error => {
            console.error('Error fetching data:', error);
            document.getElementById('loading').innerHTML = 'Error loading data. Please try again later.';
        });
}

// Function to display asteroids in table and chart
function displayAsteroids(asteroids) {
    const table = document.getElementById('asteroidTable');
    const tableBody = document.getElementById('tableBody');
    const loading = document.getElementById('loading');
    const noData = document.getElementById('noData');

    loading.style.display = 'none';

    if (asteroids.length === 0) {
        noData.style.display = 'block';
        table.style.display = 'none';
        return;
    }

    noData.style.display = 'none';
    table.style.display = 'table';
    tableBody.innerHTML = ''; // Clear table

    asteroids.forEach(asteroid => {
        const row = tableBody.insertRow();
        const hazardousBadge = asteroid.isHazardous === 'Yes' 
            ? '<span class="badge bg-danger">Yes</span>' 
            : '<span class="badge bg-success">No</span>';
        
        row.innerHTML = `
            <td>${asteroid.name}</td>
            <td>${asteroid.closeApproachDate}</td>
            <td>${asteroid.distanceKm}</td>
            <td>${asteroid.sizeKm}</td>
            <td>${hazardousBadge}</td>
        `;
    });

    // Create chart: Bar chart of distances (scaled for readability)
    const ctx = document.getElementById('distanceChart').getContext('2d');
    const chart = Chart.getChart('distanceChart'); // Destroy if exists
    if (chart) chart.destroy();

    new Chart(ctx, {
        type: 'bar',
        data: {
            labels: asteroids.map(a => a.closeApproachDate),
            datasets: [{
                label: 'Distance (Million km)',
                data: asteroids.map(a => parseFloat(a.distanceKm.replace(/,/g, '')) / 1000000),
                backgroundColor: asteroids.map(a => a.isHazardous === 'Yes' ? 'rgba(220, 53, 69, 0.6)' : 'rgba(40, 167, 69, 0.6)'),
                borderColor: asteroids.map(a => a.isHazardous === 'Yes' ? 'rgba(220, 53, 69, 1)' : 'rgba(40, 167, 69, 1)'),
                borderWidth: 1
            }]
        },
        options: {
            responsive: true,
            scales: {
                y: {
                    beginAtZero: true,
                    title: { display: true, text: 'Distance (Million km)', color: '#fff' }
                },
                x: {
                    title: { display: true, text: 'Date', color: '#fff' }
                }
            },
            plugins: {
                legend: { labels: { color: '#fff' } },
                title: { display: true, text: 'Asteroid Close Approaches', color: '#fff', font: { size: 16 } }
            }
        }
    });
}

// Load data on page load
document.addEventListener('DOMContentLoaded', fetchAsteroids);
