This is perhaps a slightly more advanced method for populating your prototypes but is super valuable and time-saving if you can use it. This is one of the more helpful but undocumented features of the GDS kit that can be incredibly powerful. Of course all the sample HTML is just for reference.

Folder Structure

Ensure your project structure follows the GDS Prototype Kit standards

your-project/
│
├── app/
│   ├── assets/
│   ├── views/
│   │   ├── services.html        # List of services
│   │   ├── service-detail.html  # Detailed service page
│   ├── data/
│   │   └── data.json            # Service data file
│   └── routes.js                # Routes configuration
│
└── package.json

Step 1: Create the data.json File

Place the JSON data in the app/data/ folder. This file contains the service data that will be accessed by the Nunjucks templates. It’s probably worth validating your JSON using a validator tool to check for any errors before you start.

Example app/data/data.json:

[
  {
	"useCase": "Land Use Change Monitoring",
	"userGroup": [
	  { "type": "Government Officials", "count": 20 },
	  { "type": "Environmental Analysts", "count": 15 }
	],
	"description": "A system to monitor land use changes using satellite imagery and geospatial data.",
	"inputArtefact": [
	  { "name": "Satellite Imagery", "url": "<https://example.com/satellite-imagery>" },
	  { "name": "Geospatial Data", "url": "<https://example.com/geospatial-data>" }
	],
	"meansOfDelivery": ["Web Portal", "API"],
	"providerOwner": "Defra Environmental Monitoring Team",
	"outputArtefact": [
	  { "name": "Land Use Report", "url": "<https://example.com/land-use-report>" },
	  { "name": "Interactive Map", "url": "<https://example.com/interactive-map>" }
	]
  }
]

Step 2: Configure routes.js

We’ll load the JSON data in routes.js and pass it to the templates globally so that it can be accessed from any page. Update your routes file to pull in the JSON and handle the routing.

Example app/routes.js:

const serviceData = require('./data/data.json');

// Middleware to make serviceData available globally in all templates
router.use(function(req, res, next) {
  res.locals.serviceData = serviceData;
  next();
});

// Route to display all services
router.get('/services', function (req, res) {
  res.render('services.html');  // No need to pass data; serviceData is accessed directly in the template
});

// Route to display details for a specific service (accessed via query parameter)
router.get('/service-detail', function (req, res) {
  const serviceId = req.query.id;  // Get service ID from query parameters
  res.render('service-detail.html', { serviceId: serviceId });  // Pass service ID to the template
});

module.exports = router;

Explanation

Loading data.json: We load the data.json file using require() and store it in serviceData.

Middleware The middleware function makes serviceData available globally via res.locals.serviceData, allowing it to be accessed in any Nunjucks template.

/services Route: Renders the services.html template, which directly accesses serviceData to display the list of services.

/service-detail Route: Renders the service-detail.html template and passes the selected serviceId from the query string, which the template uses to display the corresponding service.

Step 3: Create a services.html Template

This page lists all services by accessing serviceData directly in the template from your JSON.

Example app/views/services.html

<!DOCTYPE html>
<html lang="en">
<head>
	<meta charset="UTF-8">
	<meta name="viewport" content="width=device-width, initial-scale=1.0">
	<title>Services</title>
	<style>
		.grid-container {
			display: grid;
			grid-template-columns: repeat(auto-fit, minmax(300px, 1fr));
			gap: 20px;
		}
		.card {
			border: 1px solid #ddd;
			padding: 20px;
			border-radius: 5px;
			background-color: #f9f9f9;
		}
	</style>
</head>
<body>

<h1>All Services</h1>

<!-- Search box to filter services -->
<input type="text" id="searchInput" placeholder="Search services..." onkeyup="filterServices()">
<br><br>

<div id="cardContainer" class="grid-container">
	<!-- Loop through all serviceData items to render cards -->
	{% for service in serviceData %}
	<div class="card">
		<h2>{{ service['useCase'] }}</h2>
		<p>{{ service['description'] | truncate(100) }}</p>

		<!-- Link to navigate to the service detail page, passing the service ID as a query parameter -->
		<a href="/service-detail?id={{ loop.index0 }}">Read more</a>
	</div>
	{% else %}
	<p>No services available.</p>
	{% endfor %}
</div>

<script>
// Function to filter services based on search input
function filterServices() {
	const input = document.getElementById('searchInput').value.toLowerCase();
	const cards = document.querySelectorAll('.card');

	cards.forEach(card => {
		const title = card.querySelector('h2').textContent.toLowerCase();
		const description = card.querySelector('p').textContent.toLowerCase();

		if (title.includes(input) || description.includes(input)) {
			card.classList.remove('hidden');
		} else {
			card.classList.add('hidden');
		}
	});
}
</script>

</body>
</html>

Explanation

Search Box: The user can type in the search box, and the filterServices() function filters the services in real-time.

Rendering Services: We loop through serviceData using {% for service in serviceData %}, rendering each service's title (useCase) and description.

Service Details Link: The service ID (from loop.index0) is passed as a query parameter to the service detail page.

Step 4: Create the service-detail.html Template

This page shows detailed information about a specific service based on the serviceId passed from the query string.

Example app/views/service-detail.html:

<!DOCTYPE html>
<html lang="en">
<head>
	<meta charset="UTF-8">
	<meta name="viewport" content="width=device-width, initial-scale=1.0">
	<title>{{ serviceData[serviceId]['useCase'] }} Details</title>
</head>
<body>

<h1>{{ serviceData[serviceId]['useCase'] }}</h1>

<div class="details-container">
	<p><strong>Description:</strong> {{ serviceData[serviceId]['description'] }}</p>

	<p><strong>Users:</strong></p>
	<ul>
		{% for group in serviceData[serviceId]['userGroup'] %}
		<li>{{ group['count'] }} {{ group['type'] }}</li>
		{% endfor %}
	</ul>

	<p><strong>Input Artefacts:</strong></p>
	<ul>
		{% for artefact in serviceData[serviceId]['inputArtefact'] %}
		<li><a href="{{ artefact['url'] }}">{{ artefact['name'] }}</a></li>
		{% endfor %}
	</ul>

	<p><strong>Means of Delivery:</strong> {{ serviceData[serviceId]['meansOfDelivery'] | join(', ') }}</p>

	<p><strong>Provider/Owner:</strong> {{ serviceData[serviceId]['providerOwner'] }}</p>

	<p><strong>Output Artefacts:</strong></p>
	<ul>
		{% for artefact in serviceData[serviceId]['outputArtefact'] %}
		<li><a href="{{ artefact['url'] }}">{{ artefact['name'] }}</a></li>
		{% endfor %}
	</ul>

	<a href="/services">Back to all services</a>
</div>

</body>
</html>

Explanation:

Accessing Data with serviceId: The specific service is accessed from serviceData using the serviceId passed from the query string.

Rendering Details: All details of the selected service (useCase, description, userGroup, inputArtefact, etc.) are displayed on the page.