# DWEC UT07: Librerias complementarias en React.

Actualmente, las aplicaciones web, destacan por tener bien diferenciadas las capas: frontend, backend y datos. La comunicación entre el frontend (o interface) y el backend (o capa que implementa la lógica de negocio), es realizada a través de lo que se denomina APIs, generalmente del tipo REST. Esta es una tarea crítica y cotidiana.

Cuando implementamos aplicaciones usando React, para realizar este proceso de consumo podemos usar las funciones especiales que la librería ofrece: *useState* y *useEffect*.

## React Query (Tanstack Query)

**React Query** y **TanStack Query** son la misma biblioteca. "React Query" fue el nombre original, pero se cambió oficialmente a "TanStack Query" a partir de la versión 4 para reflejar que ahora es compatible con múltiples frameworks más allá de React, como Vue, Svelte y Solid.
PAra poder utilizarlo, primero tendremos que instalar la libreria en nuestro proyecto como siempre.

```bash
npm i @tanstack/react-query
```

Para dejar disponible a **React Query**, debemos envolver los componentes que necesiten consumir datos, su comportamiento o uso es similar a **context API**, en nuestro caso vamos a suponer que estamos en una aplicación pequeña, que tiene una barra de navegación (*Nav*), un banner o Hero, una Galería de imágenes (*Gallery*) y un pie o *Footer*, por lo que vamos a realizar este proceso a nivel de la definición de la aplicación.

```jsx
import { QueryClientProvider, QueryClient } from '@tanstack/react-query';

const queryClient = new QueryClient();

function App() {

  return (
    <QueryClientProvider client={queryClient}>
    <div style={{display:'flex', width:'100vw', flexDirection:'column'}}>
      <NavBar></NavBar>
      <Hero></Hero>
      <Gallery></Gallery>
      <Footer></Footer>
    </div>
    </QueryClientProvider>
  )
}
```

Con el código anterior, ya podemos hacer uso de React Query, en cualquiera de los componentes envueltos.

### Consumiendo una API

Para hacer uso de React Query, lo primero es hacer las importaciones necesarias dentro del componente, en este caso importamos el hook **useQuery**. 

`useQuery()` necesita 2 parametros:

* **queryKey**, nombre o identificador de la consulta a realizar
* **queryFn**, es la promesa encargada de procesar la consulta, importante tener en cuenta que se debe esperar a la ejecución, React Query hace eso por nosotros, entonces no debemos colocar await.

```jsx
import { useQuery } from '@tanstack/react-query';

const { isLoading, data, isError, error, isSuccess } = useQuery(
  {
    queryKey: ['productos'],
    queryFn: async () => {
      const res = await fetch('https://randomuser.me/api/');
      return res.json();
    }
  });
```

Observemos que recibimos un objeto con propiedades que desestructuramos en la misma línea, revisemos cada una de ellas:

* **isLoading**, es una variable booleana (`true`, `false`) que nos indica si la consulta está cargando datos.
* **data**, son los datos retornados por la consulta, que estarán disponibles luego de la carga, es decir, cuando *isLoading* pase a ser `false`.
* **isError**, otra variable booleana (`true`/`false`) que nos indica si hubo algún error a la hora de ejecutar la consulta.
* **error**, almacena cualquier error que haya sucedido, importante para poder dar respuesta al usuario en caso de que no haya podido realizarse la consulta.
* **isSuccess**, variable booleana que va a estar en `true` cuando la consulta sea realizada con éxito.

Además de las anteriores, existen otras variables que nos dan aún más información y que puedes consultar en la documentación oficial del paquete.

```jsx
import { useQuery } from '@tanstack/react-query';
import Product from '../Product/Product';



const Gallery = () => {
    
    const { isLoading, data, isError, error } = useQuery(
            {
                queryKey: ['productos'],
                queryFn: async () => {
                    const res = await fetch('https://apiexpress-x7sl.onrender.com/productos');
                    return res.json();
                }
            });
    
    if (isLoading)
        return (
            <>
                <h2>Cargando productos...</h2>
            </>
        )

    if (isError) {
        return (
            <>
                <h2>{error.message}</h2>
            </>
        )
    }
  return (
    <>
        <div className="grid place-items-center w-full bg-base-200">
            <div className="max-w-5xl py-24 content-center justify-center">
                <h1 className="text-4xl  text-center font-bold">Our Services</h1>
                <div className="grid mt-12 md:grid-cols-3 grid-cols-1 gap-8">
            
            {
                data?.map((producto) => {
                    return <Product key={producto._id} product={producto}></Product>
                })
            }
            </div>
         </div>
        </div>
    </>
  )
}

export default Gallery
```

<p align="center"> 
<a href="https://medium.com/@ignatovich.dm/tanstack-query-a-powerful-tool-for-data-management-in-react-0c5ae6ef037c">
<img src="./img/tanstack.webp" width="80%" height="80%" style="display: block; margin: 0 auto" />
</a><br>
<i><a href="https://medium.com/@ignatovich.dm/tanstack-query-a-powerful-tool-for-data-management-in-react-0c5ae6ef037c">Aqui podeis encontrar un ejemplo práctico mas completo</a></i>
</p>

## SWR (Stale-While-Revalidate)

Desarrollado por Vercel, es la alternativa más cercana en funcionalidad y simplicidad, ideal para fetching de datos en React.

El nombre se refiere a la estrategia de obtención de datos donde se muestran los datos obsoletos mientras se revalida en segundo plano. **SWR** es mínimo y ofrece una API simple para obtener y almacenar en caché datos remotos.

```bash
npm install swr
```

Usar **SWR** es simple. Aquí hay un ejemplo básico:

```jsx
import useSWR from 'swr';

function App() {
  const fetcher = (url) => fetch(url).then((res) => res.json());
  const { data, error, isLoading } = useSWR('/api/data', fetcher);

  if (error) return <div>Error al cargar los datos</div>;
  if (!data) return <div>Cargando...</div>;

  return <div>{data}</div>;
}
```

Una solicitud puede tener tres estados: "carga", "listo" o "error". El estado actual se puede determinar comprobando los valores de **datos**, **error** y **isLoading**. `UseSWR()` se utiliza para obtener datos de la `/api/herosection/123Endpoint`, y automáticamente maneja el almacenamiento en caché, la revalidación y el manejo de errores por usted. La función de retención es una devolución de llamada que se define para recuperar datos.

<p align="center"> 
<a href="https://swr.vercel.app/docs/getting-started">
<img src="./img/swr.avif" width="80%" height="80%" style="display: block; margin: 0 auto" />
</a><br>
<i><a href="https://swr.vercel.app/docs/getting-started">Aqui podeis encontrar un ejemplo de la página oficial</a></i>
</p>