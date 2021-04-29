# Creacionales

## Singleton


    // class Singleton
    class DragonQueConcedeDeseos
      private static DragonQueConcedeDeseos INSTANCE = new DragonQueConcedeDeseos()

      private constructor
        // no hace nada

      method static instance()
        return INSTANCE

      method concederDeseo(List<Esfera> esferas, Deseo deseo)
        ....

    // Cualquier otro componente que use al dragón
    class GuerreroZ
      method cumplirDeseo()
        List<Esfera> esferas = recolectarEsferas()
        Deseo deseo = pensarDeseo()
        DragonQueConcedeDeseos.getInstance().concederDeseo(esferas, deseo)


### Otro Ejemplo

    class Recomendador
      private static Recomendador INSTANCE = new Recomendador()
      private Estacion estacion;

      private constructor
        // no hace nada


      method configurarEstacion(Estacion estacion)
        this.estacion = estacion

      method Atuendo recomendar(List<Prenda> prendasPosibles)
        ....
        return new Atuendo(...)

¿Cómo se usaría?

    // Problema 1: propenso a error al tester

    test si_le_doy_un_pantalon_y_estoy_en_invierno_prefiere_ropa_larga()
      Recomendador.configurarEstacion(Estacion.INVIERNO)
      assert Recomendador.recomendar(....).contains(....)

    test si_le_doy_un_pantalon_y_estoy_en_verano_prefiere_ropa_corta()
      Recomendador.configurarEstacion(Estacion.VERANO)
      assert Recomendador.recomendar(....).contains(....)

    test otro_test_en_que_me_olvide_de_configurar_la_estacion()
      assert Recomendador.recomendar(....).contains(....) // ¡andá a saber que pasa aca, depende del orden!

    // Problema 2: no se puede generar dos recomendaciones al mismo tiempo

      method recomiendaLoMismo(estacion1, estacion2, prendas)
        Recomendador.configurarEstacion(estacion1)
        Recomendador.configurarEstacion(estacion2)

        // ¿WTF? Hay que reordanar el código con cuidado :/
        // Al menos es evidente... pero podría ser menos evidente
        return Recomendador.recomendar(prendas) == Recomendador.recomendar(prendas)


En todos los casos podríamos corregir el problema teniendo más cuidado o reordenando el código, pero es más fácil si tenemos múltiples instancias:

    // Problema 1

    test si_le_doy_un_pantalon_y_estoy_en_invierno_prefiere_ropa_larga()
      recomendador = new Recomendador(Estacion.INVIERNO)
      assert Recomendador.recomendar(....).contains(....)

    test si_le_doy_un_pantalon_y_estoy_en_verano_prefiere_ropa_corta()
      recomendador = new Recomendador(Estacion.VERANO)
      assert Recomendador.recomendar(....).contains(....)

    test otro_test_en_que_me_olvide_de_configurar_la_estacion()
      // ¡ni compila!
      assert recomendador.recomendar(....).contains(....)

    // Problema 2

      method recomiendaLoMismo(estacion1, estacion2, prendas)
        recomendador1 = new Recomendador(estacion1)
        recomendador2 = new Recomendador(estacion2)

        // ¡no hay ambigüedad!
        return recomendador1.recomendar(prendas) == recomendador2.recomendar(prendas)


## Builder

Queremos crear una bebida, siguiendo una forma de construcción lógica que
quizás no responde a como está constituida internamente (independizar la creación de su representación)

¿Por qué?
 - queremos dar una amplia variedad de opciones de configuración
 - queremos tener una representacíón mutable de algo inmutable
 - queremos crear en pasos


### Ejemplo 1: Bebidas en Starbucks

    BebidaBuilder builder = new BebidaBuilder()
    builder.conCafe()
    builder.conChocolate(NEGRO)
    builder.caliente()
    builder.conCanela()
    builder.tamaño(M)
    Bebida bebida = builder.crearBebida()


    Bebida bebida = new BebidaBuilder()
      .conCafe()
      .conChocolate(NEGRO)
      .caliente()
      .conCanela()
      .tamaño(M)
      .crearBebida()

### Ejemplo 2: ¡Muchas opciones de configuración!

    // Volvió el Recomendador. Nos damos cuenta de que tiene una gran cantidad de parámetros de configuración:
    class Recomendador
      private estacion
      private formalidad
      private modo // un enum de modos de sugerencia que puede ser LAS_MAS_RAPIDA_DE_CALCULAR, LA_MEJOR, LA_QUE_MENOS_MEMORIA_NECESITE, etc
      private evitarGorro
      private prendasAnteriores
      private gustosDeAmigues
      private coloresDeModa
      private ....


    // Crear un recomendador se volvió engorroso:

    // Opción 1: usar un constructor con muchos parámetros
    new Recomendador(INVIERNO, FORMAL, LA_MAS_RAPIDA, true, [...], [...], [ROJO, BORDO, NEGRO], true, false, 0, 0, 1.3.....)

    // Opción 1: usar muchos setters
    recomendador = new Recomendador() // se crea inconsistente :(
    recomendador.setEstacion(INVIERNO)
    recomendador.setFormalidad(FORMAL)
    // etc... y ojalá no nos olvidemos de nada :/

    Alternativa: Builder

    class RecomendadorBuilder
      // notar que tenemos:
      //  defaults
      //  validaciones
      //  atributos nuevos
      //  atributos calculados
      //  opciones que quedan ocultas
      private estacion
      private formalidad = INFORMAL
      private modo = LA_MEJOR
      private evitarGorro = false
      private coloresDeModa
      private persona

      method conEstacion(estacion)
        this.estacion = estacion

      // estos dos métodos permiten abstraernos parcialmente de que es un enum

      method formal()
        this.formalidadd = FORMAL

      method informal()
        this.formalidadd = INFORMAL

      method conFormalidad(formalidad)
        this.formalidad = formalidad

      // estos dos nos abstraern del enum subyacente
      // además introducen validaciones

      method calcularRapido()
        if this.modo == LA_QUE_MENOS_MEMORIA_NECESITE throw 'no podes ahorrar memoria Y velocidad'
        this.modo = LAS_MAS_RAPIDA_DE_CALCULAR

      method calcularConPocaMemoria()
        if this.modo == LAS_MAS_RAPIDA_DE_CALCULAR throw 'no podes ahorrar memoria Y velocidad'
        this.modo = LA_QUE_MENOS_MEMORIA_NECESITE

      method evitarGorro()
        evitarGorro = true

      method para(persona)
        this.persona = persona

      method crearRecomendador()
        new Recomendador(
          estacion,
          formalidad,
          modo,
          evitarGorro,
          persona.prendasAnteriores()
          personas.gustosDeAmigues(),
          ...) // es un bardo, pero lo hacemos una sola vez



    // Discusión: mutabilidad, estado inconsistente

### Ejemplo 3: Fechas de Java

## Factory Method


    pizzeria = new Pizzeria()
    // queremos crear la pizza del lugar
    pizzeria.crearPizza()

    class Pizzeria
      method crearPizza()
        return new Pizza()

    class Pizzeria
      method crearPizza()
        pizza = new Pizza()
        pizza.preparar()
        pizza.cocinar()
        pizza.cortar()
        pizza.empaquetar()
        return pizza


    abstract class Pizzeria
      method crearPizza
      pizza = this.instanciarPizza()
      pizza.preparar()
      pizza.cocinar()
      pizza.cortar()
      pizza.empaquetar()
      return pizza

    class PizzeriaDeGerli
      method instanciarPizza
        return new PizzaGerli()

    class PizzeriaDePalermo
      method instanciarPizza
        return new PizzaPalermo()




