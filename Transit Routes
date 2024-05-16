import streamlit as st
import requests
import folium
from openai import OpenAI
from streamlit_folium import folium_static

client = OpenAI(api_key="my-open-api-key")
API_KEY = "my-google-api-key"

def calculate_route(origin, destination):
    url = f"https://maps.googleapis.com/maps/api/directions/json?origin={origin}&destination={destination}&mode=transit&key={API_KEY}"
    response = requests.get(url).json()
    if response["status"] != "OK":
        st.error("Failed to calculate route. Please check your input and API key.")
        return

    route_polyline = response["routes"][0]["overview_polyline"]["points"]
    route_coordinates = decode_polyline(route_polyline)
    m = folium.Map(location=[sum(p[0] for p in route_coordinates)/len(route_coordinates),
                             sum(p[1] for p in route_coordinates)/len(route_coordinates)],
                   zoom_start=12,
                   tiles=None)

    folium.TileLayer(
        tiles='https://mt1.google.com/vt/lyrs=m&x={x}&y={y}&z={z}',
        attr='Google',
        name='Google Maps',
        overlay=True
    ).add_to(m)

    folium.PolyLine(locations=route_coordinates, color="blue").add_to(m)
    folium_static(m)

    route_steps = response["routes"][0]["legs"][0]["steps"]
    for step in route_steps:
        st.write(step["html_instructions"])
        st.write("Duration:", step["duration"]["text"])
        st.write("Distance:", step["distance"]["text"])
        st.write("-" * 50)

with st.form(key = "chat"):

    menu_selection = st.sidebar.selectbox(
        "Choose an option!",
        ("Air Quality Index", "Air Quality Solutions", "Public Transit Route")
    )

def decode_polyline(polyline_str):
    index, lat, lng = 0, 0, 0
    coordinates = []
    shift = 0
    result = 0
    while index < len(polyline_str):
        b, result, shift = 0, 0, 0
        while True:
            b = ord(polyline_str[index]) - 63
            index += 1
            result |= (b & 0x1f) << shift
            shift += 5
            if b < 0x20:
                break
        dlat = ~(result >> 1) if result & 1 else (result >> 1)
        lat += dlat

        shift = 0
        result = 0
        while True:
            b = ord(polyline_str[index]) - 63
            index += 1
            result |= (b & 0x1f) << shift
            shift += 5
            if b < 0x20:
                break
        dlng = ~(result >> 1) if result & 1 else (result >> 1)
        lng += dlng

        coordinates.append((lat * 1e-5, lng * 1e-5))

    return coordinates

st.title("Optimized Public Transit Route 🚌")

origin = st.text_input("Enter origin City or coordinates:")
destination = st.text_input("Enter City destination or coordinates:")

if st.button("Calculate Route"):
    if origin and destination:
        calculate_route(origin, destination)
    else:
        st.warning("Please fill in both origin and destination fields.")
