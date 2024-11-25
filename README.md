<div align='center'>
  <img height="60" src="https://user-images.githubusercontent.com/11247099/145112184-a9ff6727-661c-439d-9ada-963124a281f7.png">
  <img height="50" src="https://static-00.iconduck.com/assets.00/file-type-jest-icon-1854x2048-2w6gjixc.png">
    <img height="60" src="https://www.emojiall.com/images/240/emojione/1f419.png">
  <h1>Resumen y detalles de Testing en React</h1>

  <sup>Deja tu :star: si te gusta el proyecto.</sup>

</div>

## **Índice**

- [**Índice**](#índice)
- [**¿Cuándo y por qué hay que mockear un módulo?**](#cuándo-y-por-qué-hay-que-mockear-un-módulo)
  - [Cuándo Mockear un Módulo](#cuándo-mockear-un-módulo)
  - [Por Qué Mockear un Módulo](#por-qué-mockear-un-módulo)
  - [Ejemplo Práctico](#ejemplo-práctico)
- [**Mockear Store con Vite**](#mockear-store-con-vite)
- [**Mockear un módulo**](#mockear-un-módulo)
- [**Explicación de algún ejemplo**](#explicación-de-algún-ejemplo)
- [**Ejemplo de mockear una utilidad**](#ejemplo-de-mockear-una-utilidad)
- [**Simular eventos**](#simular-eventos)
- [**Errores**](#errores)
  - [Error: The given element does not have a value setter](#error-the-given-element-does-not-have-a-value-setter)
- [**Funcionamiento de screen.debug()**](#funcionamiento-de-screendebug)



## **¿Cuándo y por qué hay que mockear un módulo?**
Mockear un módulo es una práctica común en las pruebas de software, especialmente en el desarrollo de aplicaciones con componentes interdependientes. Aquí te explico cuándo y por qué deberías hacerlo:

### Cuándo Mockear un Módulo

1. **Dependencias Externas**:
   - **APIs y Servicios Web**: Cuando tu código depende de servicios externos, como APIs, es útil mockear estos servicios para evitar llamadas reales durante las pruebas.
   - **Bases de Datos o datos del contexto**: Si tu aplicación interactúa con una base de datos, puedes mockear las operaciones de la base de datos para evitar modificar datos reales.

2. **Comportamientos No Deterministas**:
   - **Funciones Asíncronas**: Mockear funciones que realizan operaciones asíncronas (como fetch) te permite controlar el flujo de la prueba y simular diferentes respuestas.
   - **Fechas y Tiempos**: Mockear funciones relacionadas con el tiempo (como `Date.now()`) te permite probar comportamientos dependientes del tiempo de manera consistente.

3. **Componentes Complejos**:
   - **Hooks Personalizados**: Si tu componente depende de hooks personalizados que manejan lógica compleja, puedes mockear estos hooks para simplificar las pruebas.
   - **Componentes de Terceros**: Mockear componentes de bibliotecas externas te permite enfocarte en probar tu propio código sin preocuparte por el comportamiento de componentes externos.

### Por Qué Mockear un Módulo

1. **Aislamiento de Pruebas**:
   - **Independencia**: Mockear módulos permite aislar el componente o función que estás probando, asegurando que las pruebas no se vean afectadas por el comportamiento de dependencias externas.
   - **Control**: Te da control total sobre el entorno de prueba, permitiéndote simular diferentes escenarios y respuestas.

2. **Eficiencia**:
   - **Velocidad**: Las pruebas que no dependen de servicios externos o bases de datos reales son más rápidas, lo que mejora la eficiencia del ciclo de desarrollo.
   - **Consistencia**: Mockear elimina la variabilidad introducida por factores externos, como la latencia de red o el estado de la base de datos, asegurando resultados de prueba consistentes.

3. **Seguridad**:
   - **Datos Sensibles**: Evita el uso de datos reales y sensibles en las pruebas, reduciendo el riesgo de exposición de información confidencial.
   - **Integridad de Datos**: Mockear operaciones de base de datos previene la modificación accidental de datos reales durante las pruebas.

### Ejemplo Práctico

Supongamos que tienes un componente que hace una llamada a una API para obtener datos de usuario:

```typescript
// UserComponent.tsx
import React, { useEffect, useState } from 'react';
import { fetchUserData } from './api';

const UserComponent: React.FC = () => {
  const [userData, setUserData] = useState(null);

  useEffect(() => {
    fetchUserData().then(data => setUserData(data));
  }, []);

  return (
    <div>
      {userData ? <div>{userData.name}</div> : <div>Loading...</div>}
    </div>
  );
};

export default UserComponent;
```

Para probar este componente sin hacer una llamada real a la API, puedes mockear el módulo `api`:

```typescript
// UserComponent.test.tsx
import React from 'react';
import { render, screen, waitFor } from '@testing-library/react';
import '@testing-library/jest-dom';
import { describe, it, expect, vi } from 'vitest';
import UserComponent from './UserComponent';
import { fetchUserData } from './api';

// Mock del módulo api
vi.mock('./api', () => ({
  fetchUserData: vi.fn(),
}));

describe('UserComponent', () => {
  it('displays user data after fetching', async () => {
    const mockUserData = { name: 'John Doe' };

    //TODO: Tengo que comprobar si este mock realmente funciona
    (fetchUserData as unknown as ReturnType<typeof vi.fn>).mockResolvedValue(mockUserData);

    render(<UserComponent />);

    await waitFor(() => expect(screen.getByText('John Doe')).toBeInTheDocument());
  });
});
```

En este ejemplo, el mock del módulo `api` permite controlar la respuesta de `fetchUserData`, asegurando que la prueba sea rápida, consistente y no dependa de una llamada real a la API.

**[⬆ Volver a índice](#índice)**

---


## **Mockear Store con Vite**  

```js
    import useInvoiceStoreContext from "@/hooks/use-invoice-store-context";
    ...
    vi.mock("@/hooks/use-invoice-store-context");
    ...
    (useInvoiceStoreContext as unknown as ReturnType<typeof vi.fn>).mockReturnValue({
      state: mockContext,
    });
```

Este código está utilizando TypeScript para hacer un mock del hook `useInvoiceStoreContext` con Vitest. Aquí tienes un desglose de lo que hace cada parte:

1. **`useInvoiceStoreContext as unknown as ReturnType<typeof vi.fn>`**:
   - **`useInvoiceStoreContext`**: Es el hook que se está mockeando.
   - **`as unknown`**: Se está utilizando para cambiar temporalmente el tipo de `useInvoiceStoreContext` a `unknown`. Esto es necesario porque TypeScript no permite cambiar directamente el tipo de una función.
   - **`as ReturnType<typeof vi.fn>`**: Después de cambiar el tipo a `unknown`, se está cambiando nuevamente el tipo a lo que devuelve `vi.fn()`. Esto le dice a TypeScript que `useInvoiceStoreContext` ahora debe ser tratado como una función mockeada.

2. **`.mockReturnValue({ state: mockContext })`**:
   - **`.mockReturnValue`**: Es un método de Vitest que se utiliza para especificar el valor que debe devolver la función mockeada cuando se llama.
   - **`{ state: mockContext }`**: Es el valor que se devolverá cuando se llame a `useInvoiceStoreContext`. En este caso, está devolviendo un objeto con una propiedad `state` que contiene `mockContext`.

En resumen, este código está configurando el hook `useInvoiceStoreContext` para que devuelva un valor específico (`mockContext`) cuando se llame durante las pruebas. Esto permite controlar el estado del contexto en los tests y verificar cómo el componente `InvoicePaymentTotal` se comporta con diferentes estados.

¿Te gustaría más detalles sobre alguna parte en particular?

**[⬆ Volver a índice](#índice)**

---

## **Mockear un módulo**
Módulo original:
```js
import useInvoiceImportAzure from "@/connects/invoices-import-azure.connect";
```
```js
const useInvoiceImportAzure = () => {
    ...
  const fetchInvoiceImportAzure = async (...) => {
    ...
  }
  return { fetchInvoiceImportAzure };
};

export default useInvoiceImportAzure;
```
Podemos mockear un módulo con de forma **básica sin implementación determinada** o **detallada con implementación personalizada**.
1. **Mock básico sin implementación personalizada:**
```js
vi.mock("@/connects/invoices-import-azure.connect");
```
- **Mock automático**: Este enfoque crea un mock automático del módulo completo. Vitest intentará mockear todas las exportaciones del módulo sin ninguna configuración adicional.
- **Menos control**: No tienes control sobre cómo se mockean las funciones individuales dentro del módulo. Esto puede ser suficiente para pruebas simples, pero puede no ser adecuado si necesitas un comportamiento específico.
- **Simplicidad**: Es más rápido y sencillo de configurar, pero a costa de la flexibilidad y el control detallado.
- **Cuándo Usarlo**: Cuando solo necesitas verificar que una función del módulo se llama, sin importar su implementación específica.
Si el módulo tiene pocas funciones y no necesitas un control detallado sobre cada una.
Para pruebas simples donde el comportamiento específico de las funciones mockeadas no es crítico.

2. **Mock detallado con implementación personalizada:**
```js
vi.mock("@/connects/invoices-import-azure.connect", () => ({
  __esModule: true,
  default: vi.fn(() => ({
    fetchInvoiceImportAzure: vi.fn(),
  })),
}));
```
- **Control total**: Este enfoque te permite definir exactamente cómo se debe comportar el módulo mockeado. En este caso, estás especificando que el módulo tiene una exportación por defecto (default) que es una función mock, y dentro de esa función mock, fetchInvoiceImportAzure también es una función mock.
- **Flexibilidad**: Puedes ajustar el comportamiento de cada función mock según sea necesario para tus pruebas. Por ejemplo, puedes definir diferentes implementaciones para fetchInvoiceImportAzure en diferentes tests.
- **Compatibilidad con ES Modules**: La propiedad __esModule: true asegura que el mock se maneje correctamente como un módulo ES.  
  
**Ejemplo:**
```js
// HOOK PERSONALIZADO

const useCustomHook = () => {
  const [data, setData] = useState<string>('Initial Data');

  const fetchData = () => {
    setData('Fetched Data');
  };

  return { data, fetchData };
};

export default useCustomHook;
```
```js
// COMPONENTE

import useCustomHook from './useCustomHook';

const MyComponent: React.FC = () => {
  const { data, fetchData } = useCustomHook();

  return (
    <div>
      <div>{data}</div>
      <button onClick={fetchData}>Fetch Data</button>
    </div>
  );
};

export default MyComponent;
```
**Pruebas con Mock Básico**  
En este caso, solo verificamos que la función fetchData se llama cuando se hace clic en el botón.
```js
import MyComponent from './MyComponent';
import useCustomHook from './useCustomHook';

// Mock básico del hook
vi.mock('./useCustomHook');

describe('MyComponent', () => {
  it('calls fetchData when button is clicked', () => {
    const mockFetchData = vi.fn(); //Este el el mock de la función que nos retorna useCustomHook
    (useCustomHook as as unknown as ReturnType<typeof vi.fn>).mockReturnValue({ data: 'Initial Data', fetchData: mockFetchData });

    render(<MyComponent />);

    fireEvent.click(screen.getByText('Fetch Data'));

    expect(mockFetchData).toHaveBeenCalled();
  });
});
```

**Pruebas con Mock Detallado**  
En este caso, controlamos el comportamiento del hook para simular diferentes escenarios.
```js
// MyComponent.test.tsx
import React from 'react';
import { render, screen, fireEvent } from '@testing-library/react';
import '@testing-library/jest-dom';
import { describe, it, expect, vi } from 'vitest';
import MyComponent from './MyComponent';
import useCustomHook from './useCustomHook';

// Mock detallado del hook
vi.mock('./useCustomHook', () => ({
  __esModule: true,
  default: vi.fn(() => ({
    data: 'Initial Data',
    fetchData: vi.fn(),
  })),
}));

describe('MyComponent', () => {
  it('calls fetchData and updates data when button is clicked', () => {
    const mockFetchData = vi.fn();
    (useCustomHook as jest.Mock).mockReturnValue({ data: 'Initial Data', fetchData: mockFetchData });

    render(<MyComponent />);

    // Verifica el estado inicial
    expect(screen.getByText('Initial Data')).toBeInTheDocument();

    // Simula el clic en el botón
    fireEvent.click(screen.getByText('Fetch Data'));

    // Verifica que fetchData fue llamado
    expect(mockFetchData).toHaveBeenCalled();

    // Simula la actualización de datos
    (useCustomHook as jest.Mock).mockReturnValue({ data: 'Fetched Data', fetchData: mockFetchData });

    // Re-renderiza el componente para reflejar el nuevo estado
    render(<MyComponent />);

    // Verifica el nuevo estado
    expect(screen.getByText('Fetched Data')).toBeInTheDocument();
  });
});
```

## **Explicación de algún ejemplo**

**¿Qué hace este código?**
```js
import { useNavigate } from "react-router-dom";

vi.mock("react-router-dom", () => ({
  ...vi.importActual("react-router-dom"),
  useNavigate: vi.fn(),
}));
```

1. Importación de useNavigate:

```js
import { useNavigate } from "react-router-dom";
```

- Importa el hook ``useNavigate``, que normalmente se utiliza para realizar navegaciones programáticas dentro de una aplicación React que usa ``react-router-dom``.
2. Mock de ``react-router-dom``:

```js
vi.mock("react-router-dom", () => ({
  ...vi.importActual("react-router-dom"), // Mantiene las funcionalidades reales de otras partes del paquete.
  useNavigate: vi.fn(), // Reemplaza `useNavigate` por un mock.
}));
```
- ``vi.mock``: Indica a Vitest que debe mockear el módulo react-router-dom.
- ``vi.importActual``: Conserva todas las implementaciones reales del módulo excepto las que se sobrescriben explícitamente. Esto significa que el resto de las funcionalidades de react-router-dom (como Link, Routes, etc.) seguirán funcionando normalmente.
- ``useNavigate: vi.fn()``: Mockea específicamente el hook useNavigate y lo reemplaza con un mock de Vitest (vi.fn()), que es una función simulada que puede ser monitoreada para verificar cuántas veces se llama, con qué argumentos, etc.


**¿Qué hace este código?**
```js
const mockNavigate = vi.fn();
    (useNavigate as jest.Mock).mockReturnValue(mockNavigate);
```
1. Crea un mock de navegación: 
   ```js
   const mockNavigate = vi.fn();
   ```
   - ``vi.fn()``: Crea una función simulada (mock) que puedes observar y controlar durante el test.
     - Permite registrar cuántas veces se llama.
     - Captura los argumentos con los que se invoca.
     - Se puede configurar para devolver valores personalizados si es necesario.
2. Sobrescribe ``useNavigate``
   ```js
   (useNavigate as jest.Mock).mockReturnValue(mockNavigate);
   o
   (useNavigate as unknown as ReturnType<typeof vi.fn>).mockReturnValue(mockNavigate);
   ```
   - ``useNavigate as jest.Mock``: Indica que useNavigate ha sido mockeado y que se puede manipular su comportamiento en el test.
   - `` mockReturnValue(mockNavigate)``: Configura el mock de useNavigate para que devuelva mockNavigate cuando el componente lo invoque.
       - En lugar de devolver la implementación real de useNavigate, se usa la función simulada mockNavigate.








**[⬆ Volver a índice](#índice)**

---
## **Ejemplo de mockear una utilidad**
Tengo una función que utilizo cómo una utilidad que recibe un número el máximo de decimales que necesito. Retorna un string con el formato del número con separador de miles y dos decimales

```js
export const formatPriceDotComma = (price: number, maxDecimals: number, locale: string = "de-DE"): string => {
  return price.toLocaleString(locale, {
    minimumFractionDigits: maxDecimals,
    maximumFractionDigits: maxDecimals,
  });
};
```
Esta función la puedo utilizar en caulquier componente para formatear sus datos.  
Para tener un mock de esta función, lo puedo hacer de dos maneras:

1. **Primera Forma**
```js
vi.mock("@/utils/helpers/check-price-type", () => ({ 
   formatPriceDotComma: vi.fn(), 
}));
```
En esta versión, `formatPriceDotComma` es un mock vacío que no tiene implementación. Al llamarlo en las pruebas, simplemente registrará las llamadas que recibe pero no devolverá ningún valor específico. Esto puede ser útil cuando solo queremos comprobar que ``formatPriceDotComma`` es invocado (por ejemplo, usando .**toHaveBeenCalledWith(...)**) sin importar el valor que devuelve. Por defecto, devolverá **undefined** si no se define un valor de retorno explícito.

Ejemplo de uso:  
En esta forma, si llamas a ``formatPriceDotComma(100, 2)`` en una prueba, el valor devuelto será **undefined**. No hay formato aplicado, y el valor de retorno no se puede usar en pruebas que dependan de un resultado específico.

2. **Segunda Forma**
```js
vi.mock("@/utils/helpers/check-price-type", () => ({
  formatPriceDotComma: vi.fn((price, decimalPlaces) => {
    return `${Number(price).toFixed(decimalPlaces).replace(".", ",")}`;
  }),
}));
```
En esta versión, el mock para ``formatPriceDotComma`` tiene una implementación personalizada. La función ahora recibe parámetros (price y decimalPlaces) y devuelve un valor formateado. <u>Esto simula más de cerca el comportamiento real de ``formatPriceDotComma``</u>, donde convierte un número a un formato específico, reemplazando el punto decimal por una coma.

Ejemplo de uso  
Con esta implementación, si llamas a ``formatPriceDotComma(100, 2)``, el mock devolverá "100,00". Este enfoque es útil para pruebas donde se necesita el valor retornado formateado, permitiendo verificar los resultados que dependen de la salida formateada de la función.

**[⬆ Volver a índice](#índice)**

---

## **Simular eventos**
Aquí tenemos algunos ejemplos de cómo simular el flujo de entrada de tatos del usuario:
```js
const closeButton = screen.getByRole("button", { name: "" }); //Buscamos el botón con getByRole
fireEvent.click(closeButton); // Clicamos el botón
expect(mockSetVisible).toHaveBeenCalledWith(false); //Comprobamos que la función recibida por props se elecute
```

```js
// Simular entrada en un input
const netInput = screen.getByTestId("withholding-net-input");
await userEvent.type(netInput, "1000");

// Simula despliegue de uh selector
const select = screen.getByTestId("withholding-select-input-input");
await userEvent.click(select);

// Simula clicar opción del selector
const option = screen.getByText("Tax 1");
await userEvent.click(option);

// Verificar cálculo y formato de withholding amount
const withholdingAmountField = screen.getByTestId("total-withholding-input");
expect(withholdingAmountField).toHaveValue("150,00"); // 1000 * 15% = 150.00
```



**[⬆ Volver a índice](#índice)**

---
## **Errores**

### Error: The given element does not have a value setter
Intentado mockear la introducción de información en un selector de esta manera:
```js
const select = screen.getByTestId("pepelui-select-input-input");
fireEvent.change(select, { target: { value: "1" } });
```
El error **``Error: The given element does not have a value setter``** significa que React Testing Library no puede cambiar el valor del elemento seleccionado mediante fireEvent.change porque el elemento select no tiene un atributo value que pueda ser actualizado directamente.

Este problema suele ocurrir cuando:

1. El componente Select es un componente personalizado que no se comporta como un select HTML estándar o un input.
2. El componente utiliza alguna librería de componentes que maneja el estado del select internamente (por ejemplo, usando un pop-up o una lista desplegable), y no expone un value directamente para interactuar con él.

**Posibles Soluciones**
1. Usar ``fireEvent.click`` en lugar de ``fireEvent.change``  
Si el Select se basa en una lista desplegable que se abre al hacer clic, intenta simular el clic y seleccionar la opción deseada manualmente. Puedes encontrar el elemento de la lista usando su texto:

```js
// Hacer clic en el componente Select para abrir la lista de opciones
const select = screen.getByTestId("pepelui-select-input-input");
fireEvent.click(select);

// Seleccionar la opción deseada haciendo clic en ella (por el texto visible en llista)
const option = screen.getByText("Tax 1");
fireEvent.click(option);

// Verificar que el porcentaje correspondiente se muestra
expect(screen.getByDisplayValue("15")).toBeInTheDocument();

```

2.`` Usar userEvent ``   
``userEvent`` es más avanzado que ``fireEvent`` para interacciones complejas como la selección en un Select. Asegúrate de tener @testing-library/user-event instalado e intenta usarlo en lugar de fireEvent:
```js
import userEvent from "@testing-library/user-event";

// Abrir el selector
const select = screen.getByTestId("pepelui-select-input-input");
await userEvent.click(select);

// Hacer clic en la opción deseada
const option = screen.getByText("Tax 1");
await userEvent.click(option);

// Verificar el cambio en el valor de porcentaje
expect(screen.getByDisplayValue("15")).toBeInTheDocument();
```

**[⬆ Volver a índice](#índice)**

---

## **Funcionamiento de screen.debug()**
Este método proviene de React Testing Library y se utiliza para **mostrar en la console el estado actual del DOM de la pantalla**. Imprime una representación legible del DOM actual, que es muy útil para depurar y ver cómo está estructurado el contenido en un punto específico del test.

Puede manejar dos parámetros:  
1. El primer parámetro opcional normalmente permite especificar un nodo específico del DOM que quieras depurar (en lugar de todo el DOM). Si pasas undefined, estás diciendo que imprima todo el DOM en pantalla.  
   Para especificar un nodo específico del DOM en el primer parámetro de screen.debug(), simplemente necesitas obtener el nodo que deseas inspeccionar usando uno de los métodos de selección de React Testing Library, como ``screen.getByText``, ``screen.getByRole``, ``screen.getByTestId``, etc., y luego pasar ese nodo como el primer argumento de screen.debug().


2. Este segundo parámetro opcional establece el límite máximo de caracteres que mostrará screen.debug(). Esto es útil para casos en que el DOM es grande y deseas ver más allá del límite predeterminado (generalmente 7000 caracteres).

```js
//Obtener un nodo específico, por ejemplo un botón
const button = screen.getByRole("button", {name: /click me/i})

//Imprime solo el DOM de ese elemento específico
screen.debug(button)
```

Imprime todo el DOM y 30.000 caracteres:
```js
screen.debug(undefined, 30000)
```

**[⬆ Volver a índice](#índice)**

---


