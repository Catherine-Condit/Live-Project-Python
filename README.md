# Live Project
## Introduction
**Role:** Developer <br>
**Tech Stack:** Python, Django, Google Maps API, BeautifulSoup, JavaScript, HTML/CSS <br>

&emsp;The Wellness Map App is a web application that helps users discover, save, and manage their favorite wellness locations in nature. By integrating the Google Maps API and leveraging web scraping through BeautifulSoup, the app allows users to search for local wellness spots and view detailed information, including maps and links to mental health resources.

&emsp;Developed as part of the AppBuilder9000 initiative for [The Tech Academy](https://www.learncodinganywhere.com/), this project followed a Scrum development approach. Developers worked independently on their own apps throughout the two-week sprint while following Scrum practices such as daily stand-ups, sprint planning, and retrospectives to simulate a team environment and ensure organized project management throughout the sprint.

## Core Technologies
* Backend: Python, Django
* Frontend: HTML, CSS, JavaScript
* Web Scraping: BeautifulSoup
* Database: SQLite (Django ORM)
* Version Control: Git, GitHub
* APIs: Google Maps JavaScript API, Google Maps Geocoding API

## Key Features & Functionality
* **Interactive Wellness Map:** Using the Google Maps API, users can view wellness spots displayed dynamically on a map, allowing them to zoom in, navigate, and find local wellness resources.
* **Web Scraping for Mental Health Resources:** Integrated BeautifulSoup to scrape mental health resources from a third-party website, making them available for users to access directly from the app.
* **User Favorites:** Users can save their favorite wellness spots for easy access and manage their preferences within the app.
* **Advanced Search & Filtering:** Search for wellness spots by city, state, and tags. The app dynamically updates results based on selected filters.
* **Seamless Integration with Google Maps:** The app uses the Google Maps Geocoding and JavaScript APIs to convert addresses into geographical coordinates and display markers on the map, offering an interactive experience.
  
## User Stories 
1. [Basic App Structure](#basic-app-structure)
2. [Create Model](#create-model)
3. [Display All Items](#display-all-items)
4. [Details View](#details-view)
5. [Edit And Delete Functions](#edit-and-delete-functions)
6. [Connect to API](#connect-to-api)
7. [Parse JSON](#parse-json)
8. [BeautifulSoup Setup](#beautifulsoup-setup)
9. [Parse HTML](#parse-html)
10. [Front-End Improvements](#front-end-improvements)
11. [Save Favorites](#save-favorites)
12. [Choose Adventure](#choose-adventure)
##

*Jump To: [Introduction](#introduction),  [Key Features & Functionality](#key-features--functionality), [Display All Items](#display-all-items), [Connect to API](#connect-to-api), [BeautifulSoup Setup](#beautifulsoup-setup), [Front-End Improvements](#front-end-improvements), [Conclusion](#conclusion), [Key Learnings & Challenges](#key-learnings--challenges)*
##
### Basic App Structure 

&emsp;The first task involved laying the foundation of the app, including setting up the project and configuring Django. This initial setup was crucial for the entire application, requiring careful planning to ensure proper routing and views. I configured Django's settings.py and urls.py, then linked the app to the main Appbuilder9000 project homepage for easy access. I created base HTML templates (base.html and home.html) and added basic styling to ensure consistency in headers, footers, and content containers.

<p align=center>
<img src="https://github.com/Catherine-Condit/Live-Project-Python/blob/main/gifs/story1.gif"  width="100%" height="100%" />
</p>
<br/>

After confirming that the project was running smoothly, with all dependencies properly installed and configurations set, I proceeded to the next steps.
##

### Create Model

&emsp;In this stage, I focused on building the database model for wellness spots. I started by creating the "Add a Spot" HTML page, where users could submit a form containing fields for street, city, zip code, a state dropdown, description, a five-star rating system, and 12 pre-made, multi-selectable tags. Initially, I used checkboxes for tag selection and a dropdown for rating, but both elements appeared basic. Therefore, I enhanced the form by implementing a modern tag cloud for tag selection and radio buttons designed as stars for the rating system. I also ensured that the form was validated before submission.

<p align=center>
<img src="https://github.com/Catherine-Condit/Live-Project-Python/blob/main/gifs/story2.gif"  width="100%" height="100%" />
</p>
<br/>

&emsp;Next, I created the WellnessSpot model in Django to store all relevant data, including the full address, latitude, longitude, and other details from the user's submission. I used Django’s ORM to define the model and ran migrations to generate the database tables, setting the stage for database operations and the app’s core functionality.
```python
class WellnessSpot(models.Model):
    street = models.CharField(max_length=255)
    city = models.CharField(max_length=100, default='City Name')
    state = models.CharField(max_length=2, default='State Name')
    zip_code = models.CharField(max_length=10, default='XXXXX-XXXX')
    tags = models.ManyToManyField("WellnessMap.Tag", blank=True)  # Using string reference for Tag model
    description = models.TextField(blank=True, null=True)
    rating = models.IntegerField(default=0)  # Five-star rating
    lat = models.FloatField(null=True, blank=True)
    lng = models.FloatField(null=True, blank=True)

    def __str__(self):
        return f"{self.street}, {self.city}"

    @property
    def full_address(self):
        return f"{self.street}, {self.city}, {self.state} {self.zip_code}"

    def get_tags(self):
        return ','.join([tag.name for tag in self.tags.all()])

    @property
    def encoded_address(self):
        return urllib.parse.quote(self.full_address)
```
##

### Display All Items

&emsp;This story focused on displaying all saved wellness spots from the database. I created a new view, "Explore", that queried the database for wellness spots using Django’s ORM, enabling users to search by city, state, or tags. The results were passed to the template, which dynamically displayed wellness spot details such as address, description, rating, and tags. Before I added in the maps API funcionatlity, the search results were just shown in list format. Pagination was implemented to help users easily browse through multiple spots. After I added in the map (and thus, there was no need for the pagination necessarily), I added a new section on the explore page which listed all Wellness Spots in the database regardless of the search query further up on the page. This allowed me to keep my pagination code and apply it to the full database list for better user experience. I also included on the Explore page a reset search button, which cleared previous search results, and a search form to filter wellness spots based on user input.

<p align=center>
<img src="https://github.com/Catherine-Condit/Live-Project-Python/blob/main/gifs/story3.png"  width="100%" height="100%" />
</p>
<br/>

&emsp;To improve the user experience, I enhanced the interface by allowing users to filter results by location and tags, presenting an easy-to-navigate page.
```python
class Tag(models.Model):
    name = models.CharField(max_length=255, unique=True)

    def __str__(self):
        return self.name
```
``` python
spots = WellnessSpot.objects.all() #get all wellness spots initially
    tags = Tag.objects.all()

    #handle search filters if they exist in the request
    search_by = request.GET.get('search_by', '')
    city_query = request.GET.get('city', '')
    state_query = request.GET.get('state', '')
    tags_query = request.GET.get('tags', '')  # This will be a list

    #apply search filters to the queryset
    if search_by == 'city' and city_query:
        #ensure filters by city if city is provided
        spots = spots.filter(city__icontains=city_query)
    elif search_by == 'state' and state_query:
        #ensure filters by state if state is chosen
        spots = spots.filter(state=state_query)

        # Handle filtering by tags
    if tags_query:
        # Split the tags query string into individual tags (assuming comma-separated)
        selected_tags = tags_query.split(',')

        # Clean up any extra spaces
        selected_tags = [tag.strip() for tag in selected_tags]

        spots = spots.filter(tags__name__in=selected_tags).annotate(tag_count=Count('tags')).filter(tag_count=len(selected_tags))

    #pagination
    spots = spots.order_by('city')
    paginator = Paginator(spots, 10) #show 10 per page
    page_number = request.GET.get('page')
    spots = paginator.get_page(page_number)

    #render the template with filtered spots
    return render(request, 'WellnessMap/WellnessMap_explore.html', {
        'spots': spots,
        'states': states,
        'tags': tags,
        'search_by': search_by,
        'city_query': city_query,
        'state_query': state_query,
        'tags_query': tags_query
    })
```
##

### Details View 

&emsp;The Details view was designed to display comprehensive information about each wellness spot. I created a view to fetch individual wellness spots based on their unique identifier. In the Details template, I displayed detailed information, including the address, description, rating, and tags for the selected spot. A button allowed users to return to the Explore page, continuing their search without losing their progress.
```python
<h1 class="WellnessMap-details--h1">{{ spot.full_address }}</h1>
        <p class="WellnessMap-details--p"><strong>Description:</strong> {{ spot.description }}</p>
        <p class="WellnessMap-details--p"><strong>Rating:</strong>
            <div class="WellnessMap-details--starRating">
                {% for i in "12345" %}
                    {% if i|add:0 <= spot.rating %}
                        <span class="WellnessMap-details--star filled">&#9733;</span>
                    {% else %}
                        <span class="WellnessMap-details--star">&#9733;</span>
                    {% endif %}
                {% endfor %}
            </div>
        <p class="WellnessMap-details--p"><strong>Tags:</strong> {{ spot.tags.all|join:", " }}</p>
```
&emsp;I further refined the form validation by introducing addSpot.js, which handled dynamic tag selection and updated star ratings. The JavaScript also ensured that form elements were updated correctly as users interacted with them.
```python
document.addEventListener('DOMContentLoaded', function () {
    const stars = document.querySelectorAll('.star-rating .star');
    let ratingInput = document.querySelector('input[name="rating"]');

    stars.forEach(star => {
        star.addEventListener('click', function () {
            const value = this.previousElementSibling.value;
            ratingInput.value = value; // Set the rating value
            stars.forEach(s => {
                if (s.previousElementSibling.value <= value) {
                    s.style.color = '#ffca28';
                } else {
                    s.style.color = '#ddd';
                }
            });
        });
    });
});

// Function to toggle tag selection (for optional tags)
function toggleTagSelection(tagElement) {
    var tagName = tagElement.getAttribute('data-tag');
    var tagsInput = document.getElementById('tagsInput');
    var selectedTags = tagsInput.value ? tagsInput.value.split(',') : [];

    // If tag is already selected, remove it
    if (selectedTags.includes(tagName)) {
        selectedTags = selectedTags.filter(tag => tag !== tagName);
        tagElement.classList.remove('selected');
    } else {
        // If tag is not selected, add it
        selectedTags.push(tagName);
        tagElement.classList.add('selected');
    }

    // Update the hidden input field with the selected tags
    tagsInput.value = selectedTags.join(',');
}

// Function to validate the form
function validateForm() {
    var street = document.getElementById('street_input').value;
    var city = document.getElementById('city_input').value;
    var state = document.getElementById('state_input').value;
    var zipCode = document.getElementById('zip_code_input').value;
    var description = document.getElementById('description_input').value;
    var rating = document.querySelector('input[name="rating"]:checked');

    if (!street || !city || !state || !zipCode) {
        alert("Please fill out all required fields.");
        return false; // Prevent form submission
    }

    return true; // Allow form submission
}
```
##

### Edit And Delete Functions

&emsp;In this phase, I added functionality to edit and delete wellness spots saved in the database. Users could update the description, tags, and rating of their favorite spots. To facilitate this, I created an "editSpot.js" script to update form elements based on the existing data. Additionally, I implemented a delete feature that allowed users to remove spots from the list. The delete action was confirmed through a popup alert, ensuring that users did not accidentally delete a wellness spot.

<p align=center>
<img src="https://github.com/Catherine-Condit/Live-Project-Python/blob/main/gifs/story4.png"  width="100%" height="100%" />
</p>
<br/>

&emsp;I also continued improving the UI/UX of the app, adding new features and visual elements to enhance usability.
##

### Connect to API

&emsp;This stage focused on integrating external APIs to enhance the app’s functionality. I first connected to the Google Maps Geocoding API, enabling the app to convert full address strings into latitude and longitude coordinates. I then integrated the Google Maps JavaScript API to display an interactive map on the Explore page. Using JavaScript, I added functionality to create markers based on latitude and longitude data, showing the spot’s location on the map with a clickable info window that linked to its Details page.
```python
try:
    response = requests.get(url)
    data = response.json()

    if data['status'] == 'OK':
        lat = data['results'][0]['geometry']['location']['lat']
        lng = data['results'][0]['geometry']['location']['lng']
        return lat, lng
except Exception as e:
    print(f"Error fetching coordinates: {e}")

return None, None  # Return None if API fails
```
&emsp;I initially faced challenges with handling these external requests and parsing the returned JSON data, but I ended up successfully using Python’s requests library to fetch the data and Django views to render it on the front-end.
```python
// Function to add a marker to the map
function addMarker(lat, lng, address, id) {
    const marker = new google.maps.Marker({
        position: { lat, lng },
        map: map,
        title: address
    });

    const infoWindowContent = `
        <div class="WellnessMap-explore--customInfoWindow">
            <h3 class="WellnessMap-explore--infoWindowTitle">${address}</h3>
            <a href="/WellnessMap/spot/${id}/" class="WellnessMap-explore--infoWindowLink">View Details</a>
        </div>
    `;


    const infoWindow = new google.maps.InfoWindow({
        content: infoWindowContent
    });

    marker.addListener("click", () => {
        infoWindow.open(map, marker);
    });

    // Store marker in global array
    markers.push(marker);
}
```
##

### Parse JSON

&emsp;For the Explore page, I aimed to query the database based on the user's search criteria (city, state, and tags). Upon submission, the app would filter wellness spots based on the entered criteria and convert each relevant result into a map marker with a clickable info window. This process involved handling JSON data returned by the Google Maps API and rendering the markers dynamically on the map.
```python
def search_locations(request):
    search_by = request.GET.get('search_by', '').strip()
    city_query = request.GET.get('city', '').strip()
    state_query = request.GET.get('state', '').strip()
    tags_query = request.GET.get('tags', '').strip()

    # Start with all wellness spots
    spots = WellnessSpot.objects.all()

    # Apply search filters based on user input
    if search_by == 'city' and city_query:
        spots = spots.filter(city__icontains=city_query)
    elif search_by == 'state' and state_query:
        spots = spots.filter(state__iexact=state_query)

    # Handle tag filtering
    if tags_query:
        selected_tags = tags_query.split(',')  # Split by commas if multiple tags are provided
        selected_tags = [tag.strip() for tag in selected_tags]  # Clean up any extra spaces
        spots = spots.filter(tags__name__in=selected_tags).annotate(tag_count=Count('tags')).filter(tag_count=len(selected_tags))

    # Prepare JSON response with lat/lng
    spots_data = [
        {
            'id': spot.id, #important for linking to details page
            'name': spot.street,
            'lat': spot.lat,
            'lng': spot.lng,
            'address': spot.full_address,
        }
        for spot in spots if spot.lat and spot.lng
    ]

    # Return as JSON response
    return JsonResponse({'locations': spots_data})
```
&emsp;The process required several iterations to ensure that the search results matched the selected filters and that the map markers were properly aligned with their corresponding spots. This task involved significant debugging to transition from hardcoded map coordinates to dynamically generated ones.
<p align=center>
<img src="https://github.com/Catherine-Condit/Live-Project-Python/blob/main/gifs/story5.gif"  width="100%" height="100%" />
</p>
<br/>
##

### BeautifulSoup Setup

&emsp;I incorporated BeautifulSoup to scrape mental health resources from an external website and display them within the app. To do this, I installed the necessary libraries and set up a Python script to send an HTTP request to the target website. Once I received the content, I wrote code to parse the HTML and extract relevant information, such as links to mental health support resources. I also created a dedicated “Resources” page to display this information, ensuring that users could access helpful resources directly from the app.
```python
def search_locations(request):
    search_by = request.GET.get('search_by', '').strip()
    city_query = request.GET.get('city', '').strip()
    state_query = request.GET.get('state', '').strip()
    tags_query = request.GET.get('tags', '').strip()

    # Start with all wellness spots
    spots = WellnessSpot.objects.all()

    # Apply search filters based on user input
    if search_by == 'city' and city_query:
        spots = spots.filter(city__icontains=city_query)
    elif search_by == 'state' and state_query:
        spots = spots.filter(state__iexact=state_query)

    # Handle tag filtering
    if tags_query:
        selected_tags = tags_query.split(',')  # Split by commas if multiple tags are provided
        selected_tags = [tag.strip() for tag in selected_tags]  # Clean up any extra spaces
        spots = spots.filter(tags__name__in=selected_tags).annotate(tag_count=Count('tags')).filter(tag_count=len(selected_tags))

    # Prepare JSON response with lat/lng
    spots_data = [
        {
            'id': spot.id, #important for linking to details page
            'name': spot.street,
            'lat': spot.lat,
            'lng': spot.lng,
            'address': spot.full_address,
        }
        for spot in spots if spot.lat and spot.lng
    ]

    # Return as JSON response
    return JsonResponse({'locations': spots_data})
```
##

### Parse HTML

&emsp;Using BeautifulSoup, I parsed the HTML structure of the target website to extract the necessary mental health information. The data included valuable resources such as links to support organizations and useful articles. By utilizing BeautifulSoup’s parsing methods, I was able to capture relevant data, process it, and display it in a user-friendly format within the app.

<p align=center>
<img src="https://github.com/Catherine-Condit/Live-Project-Python/blob/main/gifs/story6.gif"  width="100%" height="100%" />
</p>
<br/>
##

### Front-End Improvements

&emsp;In this final stage, I focused on enhancing the front-end design of the app to improve the user experience. I refined the UI to create a calming and visually appealing layout, using soothing color schemes and interactive elements. CSS and JavaScript were employed to ensure a consistent and engaging experience across all pages. The app's design was tailored to match the overall theme of wellness and nature, enhancing the overall aesthetic appeal.

<div style="display: flex; justify-content: center;">
  <img src="https://github.com/Catherine-Condit/Live-Project-Python/blob/main/gifs/story4.gif" width="45%" height="200px" style="margin-right: 30px;" />
  <img src="https://github.com/Catherine-Condit/Live-Project-Python/blob/main/gifs/story7.gif" width="45%" height="200px"/>
</div>
<br/>

##

### Save Favorites

&emsp;This story involved adding a feature for users to save their favorite wellness spots. I implemented a system where users could mark spots as favorites, which would be stored in the database. A new heart-shaped icon was added to the Details page, allowing users to add or remove spots from their favorites list. I also created a dedicated "Favorites" page to display the user’s saved spots. This page included pagination and the ability to delete favorites dynamically, further enhancing the app’s personalization and usability.
```python
def Wellness_addRemoveFavorite(request, spot_id):
    spot = get_object_or_404(WellnessSpot, id=spot_id)

    if request.method == 'POST':
        action = request.POST.get('action')

        if action == 'add':
            # Check if the spot is already in the favorites
            if not Favorite.objects.filter(spot=spot).exists():
                # If not already favorited, add to favorites
                Favorite.objects.create(spot=spot)
                return JsonResponse({'success': True, 'message': 'Added to favorites.'})

        elif action == 'remove':
            favorite = Favorite.objects.filter(spot=spot).first()
            if favorite:
                favorite.delete()
                return JsonResponse({'success': True, 'message': 'Removed from favorites.'})
            else:
                return JsonResponse({'success': False, 'message': 'Favorite not found.'})

        return JsonResponse({'success': False}, status=400)


def Wellness_favoriteSpots(request):
    # Get all spots marked as favorites
    favorites = Favorite.objects.all()
    return render(request, 'WellnessMap/WellnessMap_favorites.html', {'favorites': favorites})
```
##

### Choose Adventure

&emsp;While this feature was not fully completed during the sprint, I began working on a page where users could explore virtual nature spots if they were unable to visit wellness locations in person. I created a 3D model island containing various nature spots, each with clickable annotations leading to audiovisual YouTube videos of those nature spots. Although I could not complete the full vision within the sprint, I was able to lay the groundwork for an immersive experience that could later be expanded into a full virtual reality adventure.
<p align=center>
<img src="https://github.com/Catherine-Condit/Live-Project-Python/blob/main/gifs/story8.gif"  width="100%" height="100%" />
<img src="https://github.com/Catherine-Condit/Live-Project-Python/blob/main/gifs/story9.gif"  width="100%" height="100%" />
</p>
<br/>
##

*Jump To: [Introduction](#introduction),  [Key Features & Functionality](#key-features--functionality), [Display All Items](#display-all-items), [Connect to API](#connect-to-api), [BeautifulSoup Setup](#beautifulsoup-setup), [Front-End Improvements](#front-end-improvements), [Conclusion](#conclusion), [Key Learnings & Challenges](#key-learnings--challenges)*

## Conclusion

&emsp;The Wellness Map App is a fully functional platform that combines several technologies and features to provide users with an engaging and useful tool for exploring wellness locations. Through this project, I gained valuable experience in full-stack web development, API integration, and web scraping. While there were many challenges along the way, I was able to learn and adapt quickly, applying new knowledge to create a solution that addresses a real user need. I’m excited to continue improving and expanding the app, especially with features like the "Choose Adventure" page, which is still under development.

## Key Learnings & Challenges
- Adapted to Independent Scrum Development: I adhered to Scrum methodologies, working independently but following a structured process with sprints, daily stand-ups, and retrospectives. I practiced time management, ensuring I met deadlines and iterated on features efficiently.
- Full-Stack Web Development: I gained practical experience in Django for backend development, from model creation and database management to routing and views. I learned how to implement front-end designs with HTML/CSS and connect back-end functionality seamlessly with the front-end. 
- API Integration: I integrated two external APIs into my project, including the Google Maps JavaScript and Geocoding APIs. I learned how to handle and manipulate JSON data and communicate with external servers using Python’s requests library.
- Data Handling & Web Scraping: Implementing BeautifulSoup helped me learn how to effectively scrape data from external websites and integrate it into a database-driven application.
- Front-End Design and User Experience: I focused on making the app easy to use and visually engaging, using CSS and JavaScript to enhance the design and usability.
- Version Control: This project required proficient use of Git for version control and collaboration, ensuring a streamlined development workflow.
- Problem-Solving and Debugging: Throughout the project, I faced various challenges that required creative solutions. I tackled various technical challenges, including database query optimization, API integration, and ensuring the Google Maps API worked seamlessly with dynamic data. In debugging, I learned to step through issues systematically, using logging and error handling to resolve problems effectively. Here are some examples:
  * Map Location Accuracy: Debugged discrepancies in location markers on the map by refining how the coordinates were saved in the database.
  * JSON Parsing Issues: Identified and resolved issues with the parsing of JSON data from external APIs, ensuring that all relevant data was correctly extracted and displayed.
  * Scraping Challenges: Solved issues with scraping external data by adjusting HTML parsing logic when web page structures changed during development.
  * Database Query Optimizations: Improved database queries to ensure efficient retrieval of wellness spot data for faster page loads.
  * Tag Cloud Implementation: Resolved issues with generating a tag cloud for categorizing wellness spots. I debugged the dynamic generation of the tag cloud based on user input and database content, ensuring tags displayed correctly.
  * Radio Button Issues: Debugged functionality with radio buttons used for filtering favorite wellness spots based on user selection. The issue was related to incorrect id handling, which I resolved by adjusting backhand logic and ensuring the correct filtering behavior.
- Self-Learning and Research: This project pushed me to learn new tools and concepts, including how to handle new APIs, manage dynamic maps, and scrape data from websites. Building this application from the ground up allowed me to work through various challenges, and the process was both educational and rewarding. The final product addresses a clear user need and has helped me develop a better understanding of how to approach both technical tasks and project management from start to finish.

##
*Jump To: [Introduction](#introduction),  [Key Features & Functionality](#key-features--functionality), [Display All Items](#display-all-items), [Connect to API](#connect-to-api), [BeautifulSoup Setup](#beautifulsoup-setup), [Front-End Improvements](#front-end-improvements), [Conclusion](#conclusion), [Key Learnings & Challenges](#key-learnings--challenges)*
