import requests
import os

API_KEY = os.getenv("OMDB_API_KEY", "8f9204f4")  # Use your valid API key

def get_movies_by_genre(genre, min_rating, language):
    """Fetch movies filtered by genre, rating, and language."""
    print(f"\nSearching for {genre} movies in {language} with rating >= {min_rating}...\n")
    movies = []
    page = 1

    while len(movies) < 5 and page <= 10:  # Limit pages to prevent long delays
        url = f"http://www.omdbapi.com/?s=movie&apikey={API_KEY}&type=movie&page={page}"
        response = requests.get(url)

        if response.status_code != 200:
            print("❌ Failed to connect to OMDb API.")
            return []

        data = response.json()
        if data.get("Response") == "False":
            print(f"❌ API Error: {data.get('Error', 'Unknown error')}")
            return []

        for item in data.get("Search", []):
            title = item.get('Title')
            movie_data = get_movie_data(title)
            if movie_data:
                if (genre.lower() in movie_data['Genre'].lower() and
                        movie_data['Rating'] >= min_rating and
                        language.lower() in movie_data['Language'].lower()):
                    movies.append(movie_data)
                    if len(movies) >= 5:
                        break
        page += 1

    return sorted(movies, key=lambda x: x['Rating'], reverse=True)


def get_movie_data(title):
    """Fetch full movie details by title."""
    url = f"http://www.omdbapi.com/?t={title}&apikey={API_KEY}&type=movie"
    response = requests.get(url)

    if response.status_code == 200:
        data = response.json()
        if data.get("Response") == "True":
            try:
                rating = float(data.get('imdbRating', '0') or 0)
            except ValueError:
                rating = 0.0
            return {
                'Title': data.get('Title', 'Unknown'),
                'Genre': data.get('Genre', 'Unknown'),
                'Year': data.get('Year', 'Unknown'),
                'Rating': rating,
                'Language': data.get('Language', 'Unknown')
            }
    return None


def shuffle_recommendations(recommendations):
    """Show recommendations one by one until user accepts."""
    index = 0
    while index < len(recommendations):
        movie = recommendations[index]
        print(f"\nRecommended: {movie['Title']} | Genre: {movie['Genre']} | "
              f"Year: {movie['Year']} | Rating: {movie['Rating']} | Language: {movie['Language']}")
        choice = input("Are you satisfied? (yes/no): ").strip().lower()
        if choice == 'yes':
            return
        index += 1
    print("\nNo more recommendations available.")


def main():
    """Main function: take user input and show movie recommendations."""
    genre = input("Enter preferred genre (e.g., Action, Comedy): ").strip()
    language = input("Enter preferred language (e.g., English, Hindi): ").strip()
    try:
        min_rating = float(input("Enter minimum IMDb rating (e.g., 7.5): ").strip())
    except ValueError:
        print("Invalid rating input. Please enter a number.")
        return

    recommendations = get_movies_by_genre(genre, min_rating, language)
    if recommendations:
        shuffle_recommendations(recommendations)
    else:
        print("No suitable movies found.")


if __name__ == "__main__":
    main()
