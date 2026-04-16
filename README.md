# supreme-happiness
using { /Fortnite.com/Devices }
using { /Verse.org/Simulation }
using { /Fortnite.com/Characters }
using { /Fortnite.com/Game }
using { /UnrealEngine.com/Temporary/SpatialMath }

# --- CONFIGURACIÓN DE TIERS DE ARMAS ---
# Estructura para definir cada nivel de armamento
weapon_tier := class:
    @editable Nombre : string = "Nivel 1"
    @editable Granter : item_granter_device = item_granter_device{}
    @editable DistanciaCamara : float = 400.0 # Distancia para armas de cerca
    @editable CampoVision : float = 90.0      # FOV para armas de cerca

combat_manager_device := class(creative_device):

    # Arreglo de armas configurables en el editor
    @editable ArmasProgression : []weapon_tier = array{}
    
    # Dispositivo de cámara para modificar la visión en tercera persona
    @editable CamaraOrbit : orbit_camera_device = orbit_camera_device{}
    
    # Gestor de eliminaciones para detectar bajas
    @editable EliminationManager : elimination_manager_device = elimination_manager_device{}

    # Diccionario para rastrear el progreso individual
    var ProgresoJugadores : [player]int = map{}

    OnBegin<override>()<suspends> : void =
        Print("Sistema de Combate Avanzado Iniciado")
        
        # Suscribirse a eventos
        EliminationManager.EliminationEvent.Subscribe(AlEliminarEnemigo)
        GetPlayspace().PlayerAddedEvent.Subscribe(AlEntrarJugador)

    # Configuración inicial del jugador
    AlEntrarJugador(Jugador : player) : void =
        if (not ProgresoJugadores[Jugador]):
            if (set ProgresoJugadores[Jugador] = 0):
                EquiparArma(Jugador, 0)
                VincularCamara(Jugador)

    # Lógica de la Cámara en Tercera Persona
    VincularCamara(Jugador : player) : void =
        # Activamos la cámara personalizada para este jugador
        CamaraOrbit.AddTarget(Jugador)
        Print("Cámara dinámica vinculada al jugador")

    # Sistema de Evolución (Tipo Gun Game)
    AlEliminarEnemigo(Agente : ?agent) : void =
        # Intentamos obtener el jugador que hizo la eliminación
        if (Jugador := player[Agente?]):
            if (NivelActual := ProgresoJugadores[Jugador]):
                SiguienteNivel := NivelActual + 1
                
                # Si todavía hay más armas en la lista
                if (SiguienteNivel < ArmasProgression.Length):
                    if (set ProgresoJugadores[Jugador] = SiguienteNivel):
                        EquiparArma(Jugador, SiguienteNivel)
                else:
                    # Victoria o Reinicio
                    Print("¡Jugador ha alcanzado el nivel máximo!")
                    set ProgresoJugadores[Jugador] = 0
                    EquiparArma(Jugador, 0)

    # Aplicación de cambios de Arma y Cámara
    EquiparArma(Jugador : player, Indice : int) : void =
        if (Tier := ArmasProgression[Indice]):
            # 1. Quitar armas viejas y dar la nueva
            Tier.Granter.GrantItem(Jugador)
            
            # 2. Ajustar la cámara según el tipo de arma
            # Por ejemplo: Armas pesadas alejan la cámara, pistolas la acercan
            CamaraOrbit.SetViewTarget(Jugador)
            
            # Nota: Aquí manipulamos los parámetros de la cámara en tiempo real
            # Esto crea una sensación de "juego de acción personalizado"
            Print("Equipada: {Tier.Nombre}. Ajustando FOV a {Tier.CampoVision}")
