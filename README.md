# 
import pandas as pd
import networkx as nx
import matplotlib.pyplot as plt

roster_2020 = pd.read_csv("cowboys_roster_2020.csv")
roster_2021 = pd.read_csv("cowboys_roster_2021.csv")
roster_2022 = pd.read_csv("cowboys_roster_2022.csv")
roster_2023 = pd.read_csv("cowboys_roster_2023.csv")
roster_2024 = pd.read_csv("cowboys_roster_2024.csv")
roster_2025 = pd.read_csv("cowboys_roster_2025.csv")
roster_2026 = pd.read_csv("cowboys_roster_2026.csv")

rosters = pd.concat([
    roster_2020,
    roster_2021,
    roster_2022,
    roster_2023,
    roster_2024,
    roster_2025,
    roster_2026
])

print("Rows:", len(rosters))

rosters = rosters[['season', 'team', 'player', 'position']]

print(rosters.head())

rosters = rosters.dropna(subset=['player'])

rosters = rosters.drop_duplicates(subset=['season', 'team', 'player'])

print("Rows after cleaning:", len(rosters))

g = nx.Graph() # Build the graph

for index, row in rosters.iterrows():
    g.add_node(row['player'], name=row['player'], position=row['position'])

for season, group in rosters.groupby('season'):

    players = list(group['player'])

    i = 0

    for left_player in players:
        for right_player in players[i+1:]:

            weight = g.get_edge_data(left_player, right_player, {"weight": 0})["weight"]

            g.add_edge(left_player, right_player, weight=weight + 1)

        i += 1

print("Nodes:", len(g.nodes))
print("Edges:", len(g.edges))
print("Self loops:", nx.number_of_selfloops(g))

missing_names = 0

for u in g.nodes:
    if 'name' not in g.nodes[u]:
        missing_names += 1

print("Nodes missing names:", missing_names)

top_k = 20

centrality_degree = nx.degree_centrality(g)

for u in sorted(centrality_degree, key=centrality_degree.get, reverse=True)[:top_k]:
    print(g.nodes[u]['name'], centrality_degree[u], g.degree(u))

top_players = []

for u in sorted(centrality_degree, key=centrality_degree.get, reverse=True)[:10]:
    top_players.append([
        g.nodes[u]['name'],
        g.nodes[u]['position'],
        g.degree(u),
        centrality_degree[u]
    ])

top_players_df = pd.DataFrame(
    top_players,
    columns=['Player', 'Position', 'Unique Teammates', 'Degree Centrality']
)

top_players_df

top_nodes = sorted(
    centrality_degree,
    key=centrality_degree.get,
    reverse=True
)[:20]

top_graph = g.subgraph(top_nodes)

nx.draw(
    top_graph,
    with_labels=False,
    node_size=500
)

plt.title("Network of Top 20 Dallas Cowboys Players by Degree Centrality")
plt.show()

labels = {}

for u in top_graph.nodes:
    labels[u] = g.nodes[u]['name']

nx.draw(
    top_graph,
    labels=labels,
    with_labels=True,
    node_size=500,
    font_size=8
)

plt.title("Top 20 Most Connected Dallas Cowboys Players, 2020-2026")
plt.show()
