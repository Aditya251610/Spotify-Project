# Spotify-Project
Making Spotify playlist using webscraing
# importing modules

import requests
from bs4 import BeautifulSoup
import spotipy
from spotipy.oauth2 import *

# scrap code

date = input("Which year do you want to travel to? Enter the date in this format YYYY-MM-DD:")
response = requests.get("https://www.billboard.com/charts/hot-100/" + date)

bb_web_page = response.text
soup = BeautifulSoup(bb_web_page, "html.parser")

top_100_songs_span = soup.select("li ul li h3")
top_100_songs = [song.getText().strip() for song in top_100_songs_span]

# spotify codes

sp = spotipy.Spotify(
    auth_manager=SpotifyOAuth(
        scope="playlist-modify-private", redirect_uri="http://example.com",
        client_id="b9310acde33d4d29a14865da784e9d36",
        client_secret="d82ef40707284390b2381748937d79eb",
        show_dialog=True, cache_path="token.txt"
    )
)

user_id = sp.current_user()["id"]

year = date.split("-")[0]
music_dict = {}
for song in top_100_songs:
    try:
        result = sp.search(q=f"track:{song} year:{year}", type="track")
        song_dict = result["tracks"]
        song_items = song_dict["items"]
        song = song_items[0]["external_urls"]["spotify"]
        music_dict[song] = song
    except:
        pass
    print(music_dict)

    playlist = sp.user_playlist_create(user=user_id, name=f"{date} Billboard 100", public=False)
    print(playlist)

    # Adding songs found into the new playlist
    playlist = sp.user_playlist_create(user_id, f"{date} Billboard 100", public=False, collaborative=False,
                                       description='')
    for _ in music_dict.keys():
        song_id = music_dict[_].split("/")[-1]
        add_song = sp.playlist_add_items(playlist_id=playlist["id"],
                                         items=[f'spotify:track:{song_id}'],
                                         position=None
                                         )
