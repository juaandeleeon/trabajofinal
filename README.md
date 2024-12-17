import networkx as nx
import matplotlib.pyplot as plt

def crear_red_transporte():
    """
    Crea una red de transporte urbano con múltiples rutas posibles.
    """
    # Inicializar el grafo dirigido
    # Utilizamos NetworkX para crear un grafo dirigido que modela las conexiones entre estaciones.
    G = nx.DiGraph()

    # Definimos las conexiones de la Línea 6 del metro de Madrid.
    # Cada conexión tiene:
    # - Origen: estación de salida.
    # - Destino: estación de llegada.
    # - Costo: el precio estimado entre estaciones.
    # - Tiempo: el tiempo estimado (en minutos) de viaje entre estaciones.
    linea = [
        ("Cuatro Caminos", "Nuevos Ministerios", 1.25, 3),
        ("Nuevos Ministerios", "República Argentina", 1.30, 4),
        ("República Argentina", "Avenida de América", 1.20, 3),
        ("Avenida de América", "Diego de León", 1.25, 2),
        ("Diego de León", "Manuel Becerra", 1.20, 3),
        ("Manuel Becerra", "O'Donnell", 1.15, 2),
        ("O'Donnell", "Sainz de Baranda", 1.30, 3),
        ("Sainz de Baranda", "Pacífico", 1.25, 4),
        ("Pacífico", "Méndez Álvaro", 1.20, 3),
        ("Méndez Álvaro", "Arganzuela-Planetario", 1.25, 2),
        ("Arganzuela-Planetario", "Legazpi", 1.30, 3),
        ("Legazpi", "Cuatro Caminos", 1.20, 3),  # Cerrando el circuito
    ]

    # Añadimos las conexiones al grafo.
    # Cada conexión se representa como una arista dirigida con atributos de costo y tiempo.
    for origen, destino, costo, tiempo in linea:
        G.add_edge(origen, destino, costo=costo, tiempo=tiempo)

    # Retornamos el grafo construido.
    return G


def calcular_ruta(G, origen, destino):
    """
    Calcula la mejor ruta entre dos nodos según el costo.

    :param G: Grafo de transporte.
    :param origen: Nodo de origen.
    :param destino: Nodo de destino.
    :return: Ruta y sus atributos (costo y tiempo).
    """
    try:
        # Usamos la función shortest_path de NetworkX para calcular la ruta con el menor costo.
        ruta = nx.shortest_path(G, source=origen, target=destino, weight='costo')
        
        # Calculamos el costo total de la ruta sumando los atributos 'costo' de las aristas.
        costo = sum(G[u][v]['costo'] for u, v in zip(ruta[:-1], ruta[1:]))
        
        # Calculamos el tiempo total de la ruta sumando los atributos 'tiempo' de las aristas.
        tiempo = sum(G[u][v]['tiempo'] for u, v in zip(ruta[:-1], ruta[1:]))
        
        return ruta, costo, tiempo
    except nx.NetworkXNoPath:
        # Si no hay una ruta entre las estaciones, devolvemos None.
        return None, None, None


def menu_interactivo(G):
    """
    Menú interactivo para consultar rutas y simular tránsito en la red de transporte urbano.
    """
    print("\n--- Red de Transporte Urbano ---")
    print("Paradas disponibles: ", ", ".join(G.nodes))

    while True:
        # Pedimos al usuario la estación de origen y destino.
        origen = input("\nIntroduce la parada de origen: ").strip()
        destino = input("Introduce la parada de destino: ").strip()

        # Verificamos si las estaciones ingresadas existen en el grafo.
        if origen not in G.nodes or destino not in G.nodes:
            print("Una de las paradas introducidas no existe. Inténtalo de nuevo.")
            continue

        # Calculamos la mejor ruta entre las estaciones ingresadas.
        ruta, costo, tiempo = calcular_ruta(G, origen, destino)

        if ruta:
            # Mostramos la ruta encontrada, su costo y el tiempo estimado.
            print(f"\nRuta encontrada: {' -> '.join(ruta)}")
            print(f"Costo total: {costo} euros")
            print(f"Tiempo estimado: {tiempo} minutos")

            # Simulamos el flujo de pasajeros en cada estación de la ruta.
            ocupacion_actual = 0
            for i in range(len(ruta) - 1):
                nodo_origen = ruta[i]
                nodo_destino = ruta[i + 1]
                print(f"\nEstamos en la parada {nodo_origen}, próximo destino: {nodo_destino}.")

                # Pedimos al usuario el número de pasajeros que suben y bajan.
                suben = int(input(f"¿Cuántos pasajeros suben en {nodo_origen}? "))
                bajan = int(input(f"¿Cuántos pasajeros bajan en {nodo_origen}? "))

                # Calculamos la ocupación actual en el tren.
                ocupacion_actual = max(0, ocupacion_actual - bajan) + suben

                # Calculamos el tiempo extra por subidas y bajadas de pasajeros.
                tiempo_extra = (suben + bajan) * 0.5  # 30 segundos por persona
                tiempo += tiempo_extra

                print(f"Pasajeros actuales: {ocupacion_actual}")
                print(f"Tiempo acumulado (incluyendo subidas/bajadas): {tiempo:.2f} minutos")

            print(f"\nHemos llegado al destino final: {destino}.")
            print(f"Tiempo total del trayecto: {tiempo:.2f} minutos")
        else:
            print("No existe una ruta entre las paradas seleccionadas.")

        # Preguntamos si el usuario quiere calcular otra ruta.
        continuar = input("\n¿Quieres calcular otra ruta? (s/n): ").strip().lower()
        if continuar != 's':
            break


def visualizar_red(G):
    """
    Visualiza la red de transporte con costos y tiempos en las aristas.

    :param G: Grafo de transporte.
    """
    # Usamos spring_layout para calcular posiciones de los nodos en el gráfico.
    pos = nx.spring_layout(G)
    plt.figure(figsize=(10, 8))

    # Dibujamos los nodos y aristas del grafo.
    nx.draw(G, pos, with_labels=True, node_color='lightblue', node_size=800)

    # Añadimos etiquetas en las aristas que muestran el costo y el tiempo.
    edge_labels = {
        (u, v): f"{G[u][v]['costo']} / {G[u][v]['tiempo']}"
        for u, v in G.edges
    }
    nx.draw_networkx_edge_labels(G, pos, edge_labels=edge_labels)

    # Mostramos el gráfico generado.
    plt.title("Red de Transporte Urbano (Costo / Tiempo)")
    plt.show()


# Ejecución principal
if __name__ == "__main__":
    # Creamos la red de transporte.
    red = crear_red_transporte()
    
    # Visualizamos la red en un gráfico.
    visualizar_red(red)
    
    # Activamos el menú interactivo para calcular rutas.
    menu_interactivo(red)
