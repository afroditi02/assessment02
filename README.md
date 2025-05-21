# station.py

class Station:
    def __init__(self, station_id, name, line, latitude, longitude):
        self.station_id = station_id
        self.name = name
        self.line = line
        self.latitude = latitude
        self.longitude = longitude
        self.connections = []  # (Station, time, connection_type)

    def add_connection(self, station, time, connection_type):
        self.connections.append((station, time, connection_type))

# metro_graph.py

from collections import deque
import heapq

class MetroGraph:
    def __init__(self):
        self.stations = {}

    def add_station(self, station):
        self.stations[station.station_id] = station

    def add_connection(self, id1, id2, time, connection_type):
        station1 = self.stations[id1]
        station2 = self.stations[id2]
        station1.add_connection(station2, time, connection_type)
        station2.add_connection(station1, time, connection_type)

    def shortest_path_bfs(self, start_name, end_name):
        # Shortest in terms of number of stops
        queue = deque()
        visited = set()
        path = {}
        start = self.find_station_by_name(start_name)
        end = self.find_station_by_name(end_name)
        queue.append(start)
        visited.add(start.station_id)

        while queue:
            current = queue.popleft()
            if current.station_id == end.station_id:
                return self.reconstruct_path(path, start, end)
            for neighbor, _, _ in current.connections:
                if neighbor.station_id not in visited:
                    visited.add(neighbor.station_id)
                    path[neighbor.station_id] = current.station_id
                    queue.append(neighbor)
        return []

    def fastest_path_dijkstra(self, start_name, end_name):
        # Shortest in terms of total travel time
        start = self.find_station_by_name(start_name)
        end = self.find_station_by_name(end_name)
        heap = [(0, start.station_id)]
        times = {station_id: float('inf') for station_id in self.stations}
        times[start.station_id] = 0
        path = {}

        while heap:
            current_time, current_id = heapq.heappop(heap)
            current_station = self.stations[current_id]
            if current_id == end.station_id:
                break
            for neighbor, time, _ in current_station.connections:
                new_time = current_time + time
                if new_time < times[neighbor.station_id]:
                    times[neighbor.station_id] = new_time
                    path[neighbor.station_id] = current_id
                    heapq.heappush(heap, (new_time, neighbor.station_id))

        return self.reconstruct_path(path, start, end)

    def find_station_by_name(self, name):
        for station in self.stations.values():
            if station.name.lower() == name.lower():
                return station
        raise ValueError("Station not found")

    def reconstruct_path(self, path, start, end):
        station_id = end.station_id
        route = [self.stations[station_id].name]
        while station_id != start.station_id:
            station_id = path[station_id]
            route.append(self.stations[station_id].name)
        route.reverse()
        return route

# load_stations.py

import csv
import random
from station import Station # type: ignore
from metro_graph import MetroGraph # type: ignore

def load_stations(filepath):
    graph = MetroGraph()
    station_lookup = {}

    with open(filepath, newline='', encoding='utf-8') as csvfile:
        reader = csv.DictReader(csvfile)
        for row in reader:
            station = Station(
                row['Station Code'],
                row['Station Name'],
                row['Line'],
                float(row['Latitude']),
                float(row['Longitude'])
            )
            graph.add_station(station)
            station_lookup[station.station_id] = station

    # You must manually (or programmatically) set neighbors based on line information
    # Example: if stations are sequential like NS1 - NS2 - NS3, connect NS1-NS2, NS2-NS3.

    # Random example connection setup (you need actual neighbor logic!)
    for station in graph.stations.values():
        # Fake random neighbor connection example
        if random.random() > 0.7:
            neighbor = random.choice(list(graph.stations.values()))
            if neighbor != station:
                graph.add_connection(station.station_id, neighbor.station_id, random.randint(2,8), 'same_line')

    return graph
