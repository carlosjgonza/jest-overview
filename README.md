# Jest  🧪

## 🔍 Qué es TDD (Test Driven Development)

TDD es una práctica de programación que consiste en 2 actividades, primero escribir las pruebas de la aplicación basado en que va a hacer la aplicación y luego escribir el código que pase esas pruebas. Con esta practica se espera que al principio siempre la prueba falle y luego cuando hagas el código la prueba pase (También se conoce como red-green test).

## 🤷 Por qué TDD?

- Mejor código.
	- Mejor organización, porque tienes que planear que hacer antes de escribir una sola linea de código.
	- Menos bugs, porque los bugs más evidentes las pruebas los encontrarán.

## 🤔 Qué es Jest

De acuerdo a la página oficial (https://jestjs.io/), jest es un framework orientado a pruebas con focus en la simplicidad.

## 🛠️ Instalación 

```bash
npm instal --save-dev jest
```
Para testear con React es necesario una librería desarrollada por airbnb llamada `enzyme` y su adaptador para la version de react que se está trabajando la cual nos ayuda a renderizar componentes.
De acuerdo a la documentación oficial Enzyme es un libreria utilitaria JS para react que hace más simple probar componentes React. Con ésta libreria puede manipular, modificar y simular eventos de muchas formas sobre los componentes de acuerdo a las pruebas que necesites.

``` bash
npm install --save-dev enzyme jest-enzyme enzyme-adapter-react-16
```

> Tener en cuenta que jest toma la configuración de babel del .babelrc | babel.config.json que esté en la raíz del proyecto

## 🔧 Configuración

Para configurar las pruebas con `jest` en un proyecto de react se deben seguir estos pasos

### ⭐ Configurar el enzyme para que funcione con la version de react que estamos trabajando, en este caso es la `16`

Esto se hace creando un archivo con el siguiente código
```js
//src/setupTests.js
import { configure } from  'enzyme';
import  Adapter  from  'enzyme-adapter-react-16';
configure({ adapter:  new  Adapter() });
```
Para que jest corra este archivo antes de ejecutar todas las pruebas

```json
//package.json
{
	...
	"jest": {
		"setupFilesAfterEnv": ["<rootDir>/src/setupTests.js"]
	}
}
```
> Esto también se puede hacer en un archivo jest.config.js

```js
// jest.config.js 
module.exports = {
	setupFilesAfterEnv: ["<rootDir>/src/setupTests.js"]
};
```
[Jest configuration docs](https://jestjs.io/docs/en/configuration)

## 🙌 Ya podemos empezar escribiendo el primer test


Tenemos el siguiente componente App

```js
import React from "react";

class App extends React.Component {
	render() {
		return  <h1  className="container">Hola mundo!</h1>;
	}
}
export default App;
```
La siguiente prueba
```js
//app.test.js
import  React  from  'react'
import { mount } from  'enzyme'
import  App  from  './App'

describe("Test app mount", () => {
	test("Test app container", () => {
		const  app  =  mount(<App  />)
		expect(app.find(".container").text()).toEqual("Hola mundo!")
	})
})
```
### Funciones de `jest`

- `describe`: Describe un grupo de tests
- `test|it`: para crear un test `test` y `it` básicamente son la misma cosa
- `expect`: especifica que se espera en el test

### Funciones de `enzyme`

`Enzyme` provee funciones para el montado de componentes

#### 🚣 [Shallow Rendering](https://enzymejs.github.io/enzyme/docs/api/shallow.html)

El renderizado superficial es útil para limitarse a probar un componente como una unidad, y para asegurarse de que sus pruebas no afirman indirectamente el comportamiento de los componentes secundarios, en otras palabras shallow monta el componente solo con el primer nivel de profundidad sin preocuparse por sus hijos (si el componente tiene).

```js
import { shallow } from 'enzyme';

const wrapper = shallow(<MyComponent />);
// ...
```

#### 🛳️ [Full Rendering](https://enzymejs.github.io/enzyme/docs/api/mount.html)
La representación completa de DOM,  es ideal para casos de uso en los que tiene componentes que pueden interactuar con las API de DOM o necesitan probar componentes que están envueltos en componentes de orden superior.
```js
import { mount } from 'enzyme';

const wrapper = mount(<MyComponent />);
// ...
```

#### 🌳 [Static Rendering](https://enzymejs.github.io/enzyme/docs/api/render.html)
Use la función de `render` para generar HTML desde su árbol React y analice la estructura HTML resultante.

```js
import { render } from 'enzyme';

const wrapper = render(<MyComponent />);
// ...
```

## 🕵 Query Selector en enzyme 

Para seleccionar elementos en `enzyme` es muy parecido a `document.querySelector` pero con el metodo `find` en el cual se puede buscar por `id,classname,data-attr` pero lo mejor es hacerlo con atributos data (data-...) ya que algunas librerias como `styled-components` crean hash para las clases.

Por ejemplo tenemos el siguiente componente.
```js
import  React  from  'react'

export  default (props) => {
	return (
	<div data-test="component-congrats">
		<span data-test="congrats-message">
			Congratulations! you guessed the world!
		</span>
	</div>
	)
}
```

Si queremos hacer una prueba seleccionando el `component-congrats`, se hace de la siguiente manera:

```js
//component.test.js
...
const componentCongrats = wrapper.find("[data-test='component-congrats']") 
```

Nos interesa que estos `data-test` sean visibles solo en modo de desarrollo por lo que es necesario agregar un pluggin a babel para que los quite en build de produccion

`npm install --save-exact babel-plugin-react-remove-properties`

y se agrega al `.babelrc`, eso elimina todas las propiedades data-...

```json
{
...
	"env": {
		"production": {
		"plugins": [
				"react-remove-properties"
			]
		}
	},
...
}
```

Ahora bien si queremos eliminar solo ciertas propiedades (como nuestro data-test) usamos la siguiente configuración

```json
{
...
  "env": {
    "production": {
      "plugins": [
        ["react-remove-properties", {"properties": ["data-test", "data-foo", /my-suffix-expression$/]}]
      ]
    }
  }
...
}
```

> Tener en cuenta que babel por default esta configurado en `development`,  para cambiarlo toma una variable de entorno llamada `BABEL_ENV` si no la encuentra toma el `NODE_ENV` 

#### 🗒️ Ejemplo 

Este ejercicio es un contador con dos botones uno de incremento y otro de decremento, asi como también un mensaje de error y otro de información.

```javascript
import  React, { Component } from  'react';

class  Counter  extends  Component {
	constructor(props) {
		super(props);
		this.state  = {
			counter:  0,
			error:  false
		};
	}
	render() {
		return (
		<div  data-test="component-counter">
			<h1  data-test="counter-display">The counter is currently {this.state.counter}</h1>
			<button
				data-test="increment-button"
				onClick={() => {
					if (this.state.error) this.setState({error:  false})
					this.setState({ counter:  this.state.counter  +  1 })
				}}
			>
				Increment counter
			</button>
			<button
				data-test="decrement-button"
				onClick={() =>  this.state.counter  ===  0  ?  this.setState({ error:  true }) :  this.setState({ counter:  this.state.counter  -  1 })}
			>
				Decrement counter
			</button>
			{this.state.error  &&  <p style={{color:  'red'}}>Counter should not be less than 0</p>}
		</div>
	);
	}
}

export  default  Counter;

``` 
✍️ Ahora veamos como serían las pruebas.

### ✔️ Test

```javascript
import  React  from  'react'
import { shallow } from  'enzyme'
import  Counter  from  './index'

const  findByTestArray  = (wrapper, val) => {
	return  wrapper.find(`[data-test='${val}']`)
}

const  setup  = (props  = {}, state=  null) => {
	const  wrapper  =  shallow(<Counter  {...props}/>)
	if (state) wrapper.setState(state)
	return  wrapper
}
  
describe("Test for Counter", () => {
	const  wrapper  =  setup()
	test("Render without error", () => {
		const  counterComponent  =  findByTestArray(wrapper, "component-counter")
		expect(counterComponent.length).toBe(1)
	})

		
	test("Renders increment button", () => {
		const  counterComponent  =  findByTestArray(wrapper, "increment-button")
		expect(counterComponent.length).toBe(1)
	})

	test("Renders decrement button", () => {
		const  counterComponent  =  findByTestArray(wrapper, "decrement-button")
		expect(counterComponent.length).toBe(1)
	})


	test("Renders Counter display", () => {
		const  counterComponent  =  findByTestArray(wrapper, "counter-display")
		expect(counterComponent.length).toBe(1)
	})


	test("Counter start at 0", () => {
		const  initialStateCounter  =  wrapper.state('counter')
		expect(initialStateCounter).toBe(0)
	})
		

	test("Clicking button increment counter displays", () => {
		const  counter  =  7
		const  wrapper  =  setup(wrapper, { counter })
		//find button and trigger event
		const  button  =  findByTestArray(wrapper, "increment-button")
		button.simulate('click')
		//find counter display
		const  counterDisplay  =  findByTestArray(wrapper, "counter-display")
		expect(counterDisplay.text()).toContain(counter  +  1)
	})

	test("Clicking ddecrement counter dusplay", () => {
		const  counter  =  7
		const  wrapper  =  setup(wrapper, {counter})
		const  button  =  findByTestArray(wrapper, "decrement-button")
		button.simulate("click")
		const  counterDisplay  =  findByTestArray(wrapper, "counter-display")
		expect(counterDisplay.text()).toContain(counter  -  1)
	})
})
```

Las funciones de búsqueda y montado del componente, como buena práctica se puede sacar a parte para ser reutilizadas.

```javascript
export const  findByTestArray  = (wrapper, val) => {
	return  wrapper.find(`[data-test='${val}']`)
}
```

Para el setup se pueden hacer 2, una para montar con shallow y otra con mount, todo depende del tipo de prueba que vaya a realizar

```javascript
export const  setup  = (props  = {}, state=  null) => {
	const  wrapper  =  shallow(<Counter  {...props}/>)
	if (state) wrapper.setState(state)
	return  wrapper
}
```
  
## 📒  Prop-Types

Para testear las proptypes en jest es necesario instalar la siguiente dependencia

```bash
npm install --save-dev check-prop-types
```

Se declara una función la cual puede estar contenida en un archivo de `utils` (mismo archivo del que hablamos en el punto anterior).
Luego usando el método checkPropTypes se verifica dado los propTypes del componente, las props esperadas y el nombre del componente si no retorna ningún warning.

```javascript
export  const  checkProps  = (component, conformingProps) => {
	const  propError  =  checkPropTypes(
		component.propTypes,
		conformingProps,
		'prop',
		component.name
	)

	expect(propError).toBeUndefined();
}
```

### 👷 Uso

```javascript
//en el componente Guessedwords
GuessedWords.propTypes  = {
	guessedWords:  PropTypes.arrayOf(PropTypes.shape({
		guessedWord:  PropTypes.string.isrequired,
		letterMathCount:  PropTypes.number.isRequired
	})).isRequired
}
```

```javascript
//guessedWords.test.js
const  defaultProps  = {
	guessedWords: [
		{ guessedWord:  'train', letterMathCount:  3 }
	]
}

const  setup  = (props  = {}) => {
	const  mixedProps  = {...defaultProps, ...props}
	return  shallow(<GuessedWords  {...mixedProps}/>)
}

// El resto del codigo...
// Ya nuestro método checkProps incluye el expected que necesita la prueba


test("Does not throw warning with expected props", () => {
	checkProps(GuessedWords, defaultProps)
})
```

# 🤼 toBe vs toEqual

Cuando se hacen pruebas muchas veces debemos comparar valores esperados con valores obtenidos, para hacer hay 2 método en jest (toBe y toEqual).

`toBe()`: Este es para evaluar valores primitivos como
	- number
	- string
	- boolean
	
> La comparacion es estricta, éste método generalmente se usa cuando quieres comparar objectos o arrays porque es una comparación más profunda.

`tpEqual()`: Este se usa para evaluar valores como 
	- objectos
	- arrays

# 🃏 Mock Events

En algunos casos podemos hacer pruebas verificando que nuestros métodos se ejecuten correctamente. Para eso hacemos uso de jest.fn()

```javascript
test('`guessWord` is executed when button is clicked', () => {
	// Get button in Input component and execute click
	const button = findByTestAttr(wrapper, 'submit-button');
	button.simulate('click', { preventDefault() {} });

	// Setting up mock for `guessWord`
	const guessWordMock = jest.fn();

	// Check times called mock method
	const guessWordCallCount = guessWordMock.mock.calls.length;
	expect(guessWordCallCount).toBe(1);
});
```

# 🤩 Redux

Ahi varios aspectos que podemos probar en redux, empecemos con los action creators, éstos se prueban como funciones normales.

```javascript
import { correctGuess, actionTypes } from  './'
  
describe("correctGuess", () => {
	test("Returns an action with type 'CORRECT_GUESS'", () => {
		const  action  =  correctGuess()
		expect(action).toEqual({type:  actionTypes.CORRECT_GUESS})
	})
})
```

> Ahora bien si queremos probar el comportamiento del state en redux, se puede usar el dispatch y el getState.

- Preferiblemente se debe hacer una funcion `storeFactory` que cree el store de prueba que se usará en los test 
- Sí se usan middlewares como `redux-thunk` se deben colocar en el `storefactory` para que la prueba tenga todos los componentes del store real.

```javascript
import  rootReducer  from  '../reducers'
import { createStore, applyMiddleware } from  'redux'
import { middlewares } from  '../reducers/configureStore'

export  const  storeFactory  = (initialState) => {
	const  createStoreWithMiddleware  =  applyMiddleware(...middlewares)(createStore)
	return  createStoreWithMiddleware(rootReducer, initialState)
}
```

## 📦 Componentes Redux

Para testear los componentes que usan redux se crea el store de prueba unicamente con los parametros que necesita el componente.

Existen varias pruebas que se pueden hacer:

- Se verifica que el componente este teniendo acceso a los `props` que necesita
- Se verifica que el componente este llamando a los `action creators` que necesita

OJO: Cuando queremos probar la integración de los componentes con redux una buena práctica es exportar nuestros componentes no conectados a redux para poner usar nuestro store de prueba.

```js
export class UnconnectedInput extends Component {
	...
}

export default connect(mapStateToProps, { guessWord })(UnconnectedInput);
```

Ahora bien para acceder a las props se usa el metodo `instance` del wrapper, ésto se aplica para obtener los props de redux y las actions disponibles con el componente conectado.

```js
describe("redux props", () => {
	const success =  true
	const wrapper =  setup({success})
	test("has success piece of state", () => {
		const  successProp  =  wrapper.instance().props.success
		expect(successProp).toBe(success)
	})
})
```

Otra prueba que podemos hacer con los `action creators` es verificar que son llamados con el paramétros correctos. Ésto con el componente no conectado.

```js
describe('Events', () => {
  let guessWordMock;
  let wrapper;
  const guessedWord = 'train';
  beforeEach(() => {
    // Setting up mock for `guessWord`
    guessWordMock = jest.fn();
    const props = {
      success: false,
      guessWord: guessWordMock,
    };

    // Setup Input component with mock method guessWord
    wrapper = shallow(<UnconnectedInput {...props} />);

    // Add value to input box
    wrapper.setState({ guessWordText: guessedWord });

    // Get button in Input component and execute click
    const button = findByTestAttr(wrapper, 'submit-button');
    button.simulate('click', { preventDefault() {} });
  });
  test('`guessWord` is executed with the correct parameters', () => {
    const guessWordArg = guessWordMock.mock.calls[0][0];
		// Here it's verified if our mock method is executed with the expected parameter
    expect(guessWordArg).toBe(guessedWord);
  });
  test('`guessWord` clear input box after button is clicked', () => {
    expect(wrapper.state('guessWordText')).toBe('');
  });
});
```

## 📝 Pruebas de Integración Redux

Las pruebas de integración de redux son aquellas en las que verificamos el flujo completo de redux, para verificar que todo se comporte como se espera.

En éstas pruebas lo que verificamos es que luego de despachar un action con ciertos parámetros nuestro state se modifique como esperamos.

```js
describe('no guessed words', () => {
	let store;
	const secretWord = 'party';
  const unsuccessfulGuess = 'train';
	const initialState = { secretWord };
	beforeEach(() => {
		// Create our mock store
		store = storeFactory(initialState);
	});
	test('Updates state correctly for unsuccessful guess', () => {
		// Dispatch our action
		store.dispatch(guessWord(unsuccessfulGuess));
		const newState = store.getState();
		const expectedState = {
			...initialState,
			success: false,
			guessedWords: [{
				guessedWord: unsuccessfulGuess,
				letterMatchCount: 3,
			}]
		};
		// Verify is our new state is equal to our expected state
		expect(newState).toEqual(expectedState);
	});
});
```

### 📝 Pruebas sobre Redux Saga

Hay 2 formas de probar redux-sagas, probar la funcion saga generator paso por paso ó ejecutar todo el flujo de redux-sagas y verificar el resultado.

Supongamos que tenemos las siguientes acciones:

```js
export function fetchAuthors() {
  return {
    type: 'FETCH_AUTHORS'
  }
}

export function saveAuthorsToList(authors) {
  return {
    type: 'SAVE_AUTHORS_TO_LIST',
    payload: authors
  }
}
```

Y las siguientes sagas:

```js
import { call, put, takeEvery } from 'redux-saga/effects';

import { saveAuthorsToList } from './actions';
import Api from './apis';

function* fetchAuthorsFromApi() {
  yield takeEvery('FETCH_AUTHORS', makeAuthorsApiRequest);
}

function* makeAuthorsApiRequest(){
  try {
    const authors = yield call(Api.requestAuthors);
    yield put(saveAuthorsToList(authors));  
  } catch (err) {
    yield put(saveAuthorToListError();
  }
}

export default fetchAuthorsFromApi();
```

Entonces aquí ejemplos de ambos tipos de pruebas, primero probar el paso a paso de la funcion saga generator:

1. En esta prueba estamos verificando que nuestra saga (función generadora) al llamar al next() ejecuta el método de nuestra saga.

```js
import { takeEvery } from 'redux-saga/effects';
import { fetchAuthorsFromApi, makeAuthorsApiRequest } from './sagaWithErrorHandling.js';

describe('fetchAuthorsFromApi', () => {
  const genObject = fetchAuthorsFromApi();
  
  it('should wait for every FETCH_AUTHORS action and call makeAuthorsApiRequest', () => {
    expect(genObject.next().value)
      .toEqual(takeEvery('FETCH_AUTHORS', makeAuthorsApiRequest));
  });
  
  it('should be done on next iteration', () => {
    expect(genObject.next().done).toBeTruthy();
  });
});
```

2. Ahora vamos a verificar todo el flujo de redux-saga, usando `jest.spyOn` vamos a sobreescribir  nuestro método del API `requestAuthors` y vamos a retornar un valor dummy, luego executamos nuestra saga y el valor que debe retornar debe ser nuestro dummy data, con esto verificamos que se llama la acción correcta con los valores correctos.

Del mismo modo podemos forzar en nuestro método mock que la saga responda `reject` emulando que el API respondió algún error, con ésto podemos verificar que nuestra saga llama a la acción correcta con el valor correcto.

**NOTE:** Siempre que se creen métodos mocks (`jest.fn or jest.spyOn`) ejecutar el método clear para evitar problemasa con las siguientes pruebas.

```js
import { runSaga } from 'redux-saga';
import * as api from './api';

import {
  saveAuthorsToList,
  saveAuthorsToListError
} from './actions';

import { makeAuthorsApiRequest } from './saga';

describe('makeAuthorsApiRequest', () => {
  it('should call api and dispatch success action', async () => {
    const dummyAuthors = { name: 'JK Rowling' };
    const requestAuthors = jest.spyOn(api, 'requestAuthors')
      .mockImplementation(() => Promise.resolve(dummyAuthors));
    const dispatched = [];
    const result = await runSaga({
      dispatch: (action) => dispatched.push(action),
    }, makeAuthorsApiRequest);

    expect(requestAuthors).toHaveBeenCalledTimes(1);
    expect(dispatched).toEqual([saveAuthorsToList(dummyAuthors)]);
    requestAuthors.mockClear();
  });

  it('should call api and dispatch error action', async () => {
    const requestAuthors = jest.spyOn(api, 'requestAuthors')
      .mockImplementation(() => Promise.reject());
    const dispatched = [];
    const result = await runSaga({
      dispatch: (action) => dispatched.push(action),
    }, makeAuthorsApiRequest);

    expect(requestAuthors).toHaveBeenCalledTimes(1);
    expect(dispatched).toEqual([saveAuthorsToListError()]);
    requestAuthors.mockClear();
  });
});
```

Ref: https://redux-saga.js.org/docs/advanced/Testing.html

### 🔔 Axios

Supongamos que queremos hacer pruebas de nuestro endpoints (verificar que se llaman correctamente con una acción) ó que el set dentro de nuestra acción (que espera una respuesta del API) se ejecuta con los parametros correctos.

Para eso utilizamos moxios (https://github.com/axios/moxios)

Moxios es una libreria que permite emular axios para pruebas, con ella podemos retornar datos mock cuando nuestras funciones ejecutan axios.

#### 🗒️ Ejemplo

Supongamos que tenemos la siguiente función que llama a un endpoint y la respuesta se la envía al método que recibe por parámetro.

```js
export const getSecretWord = async (setSecretWord) => {
  const response = await axios.get('http://localhost:3030');
  setSecretWord(response.data);
}
```

Usando moxios podemos crear un axios mock para retornar data dummy y verificar si nuestro método se ejecuta con la respuesta mock.

Debemos usar `install()` y `uninstall()` para sobreescribir las llamadas axios y luego usando `moxios.wait()` creamos la respuesta mock que retornará cualquier llamada axios, luego llamados nuestra acción y verificamos que la acción se ejecutó con la respuesta dummy.

```js
import moxios from 'moxios';

import { getSecretWord } from './hookActions';

describe('moxios tests', () => {
  beforeEach(() => {
    moxios.install();
  });
  afterEach(() => {
    moxios.uninstall();
  });
  test('Calls the getSecretWord callback on axios response', async () => {
    const secretWord = 'party';

    moxios.wait(() => {
      const request = moxios.requests.mostRecent();
      request.respondWith({
        status: 200,
        response: secretWord,
      });
    });

    // Create mock for callback arg
    const mockSecretWord = jest.fn();

    await getSecretWord(mockSecretWord);

    expect(mockSecretWord).toHaveBeenCalledWith(secretWord);
  });
});
```

### 🔔 Hooks

Hooks son una nueva característica en React 16.8. Estos te permiten usar el estado y otras características de React sin escribir una clase.

Con jest podemos probar estos hooks haciendo `mock` de los métodos como lo hemos hecho anteriormente, y verificar si el método se ejecuta bien y con los valores pasados.

⭐ Comencemos con el `useState`, como ya sabemos es recibe 1 parametro que es el valor inicial y retorna el valor actual y método para modificarlo.

La prueba sería asi:

1. Creamos nuestro método mock `mockSetCurrentGuess` que va a ser el método que retornará nuestro useState mock.
2. Sobreescribimos React.useState
3. Hacemos un cambio en algun componente o algo que force la ejecucción del useState
4. Verificamos que nuestro método mock se haya ejecutado con el valor correcto.

```js
describe('State controlled input field', () => {
  let mockSetCurrentGuess = jest.fn();
  let wrapper;
  beforeEach(() => {
    mockSetCurrentGuess.mockClear();
    React.useState = jest.fn(() => ["", mockSetCurrentGuess]);

    wrapper = setup({});
  });
  test('State updated with value of input box upon change', () => {
    const inputBox = findByTestAttr(wrapper, 'input-box');
    const mockEvent = { target: { value: 'train' } };
    inputBox.simulate("change", mockEvent);

    expect(mockSetCurrentGuess).toHaveBeenCalledWith('train');
  });
  test('State updated with empty value with click event', () => {
    const submitButton = findByTestAttr(wrapper, 'submit-button');
    submitButton.simulate("click", { preventDefault: () => {} });

    expect(mockSetCurrentGuess).toHaveBeenCalledWith('');
  });
});
```

⭐ Ahora bien también podemos hacer pruebas con el `useEffect`, sería asi:

1. Debemos crear nuestro setup montando nuestro componente.
2. Crear un método mock que será el que se ejecute cuando se monta el componente. (Generalmente son métodos de hooks asi que aquí reemplazamos ese método antes de montar el componente ó si es un método del prop lo pasamos como parámetro)
2. Si es un useEffect que se ejecuta cuando se monta el componente verificamos que se haya ejecutado al menos una vez.
3. Si es un useEffect que se ejecuta cuando modificamos una propiedad usamos `.setProps()` y verificamos que nuestro método mock se haya ejecutado correctamente.

🚨 Recuerda que para probar `useEffect` debes usar **mount** porque con el shallow aún no hay soporte.

```js
/**
 * Setup funtion for app component
 * @param {string} secretWord - desired secretWord state for test
 * @returns {ReactWrapper}
 */
const setup = (secretWord="party") => {
  mockGetSecretWord.mockClear();
	// Get secretWork will be executed when componente is mount
  hookActions.getSecretWord = mockGetSecretWord;

  // Use mount, because useEffect is not called on `shallow`
  return mount(<App />);
}

describe('getSecretWord calls', () => {
  test('getSecretWord calls on App mount', () => {
    setup();
    expect(mockGetSecretWord).toHaveBeenCalled();
  });
  test('getSecretWord does not update on App update', () => {
    const wrapper = setup();
    // Clear mock method because is executed on mount
    mockGetSecretWord.mockClear();

    // Wrapper.update() doesn't trigger update
    // issue in github (from https://github.com/airbnb/enzyme/issues/2091)
    wrapper.setProps();

    expect(mockGetSecretWord).not.toHaveBeenCalled();
  });
});
```

### 🎁 Adicional

Puedes probar métodos JS para validar si se ejecutan en nuestro componente ante alguna acción (por ejemplo un `console.warn`)

Para eso lo único que debemos hacer es igualar ese método JS con un método mock de jest `jest.fn` y luego verificar si nuestro mock se ejecuta correctamente.

```js
describe('Language string testing', () => {
	const mockWarn = jest.fn();
	let originalWarn;

	beforeEach(() => {
		originalWarn = console.warn;
		console.warn = mockWarn;
	});

	afterEach(() => {
		console.warn = originalWarn;
	});

	// In this case the componente call 'getStringByLanguage' to get a string
	// by the language requested and if the string doesn't exists
	// by the language the console displays warning
	test('Returns correct submit string for english', () => {
		const string = getStringByLanguage('en', 'submit', strings);
		expect(string).toBe('submit');
		expect(mockWarn).not.toHaveBeenCalled();
	});
});
```