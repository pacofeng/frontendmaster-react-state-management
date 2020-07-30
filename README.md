https://speakerdeck.com/stevekinney/react-state

### Introduction
* The main job of React is to take your application state and turn it into DOM nodes.
* There are many kinds of states:
    * Model data: The nouns in your application.
    * View/UI state: Are those nouns sorted in ascending or descending order?
    * Session state: Is the user even logged in?
    * Communication: Are we in the process of fetching the nouns from the server?
    * Location: Where are we in the application? Which nouns are we looking at?
* Or it might make sense to think about state relative to time:
    * Model state: This is likely the data in your application. This could be the item in a given list.
    * Ephemeral state: Stuff like the value of an input field that will be wiped away when you hit enter. This could be the order in which a given list is sorted.


### Class-Based State
* this.setState() is asynchronous 
`increment = () => {
    this.setState({ count: this.state.count + 1 });
    this.setState({ count: this.state.count + 1 });
    this.setState({ count: this.state.count + 1 });
    console.log(this.state.count); // 0
};`
* React is trying to avoid unnecessary render
    * the count after the increment call is 1
    * React will batch them up, figure out the result and then efficiently make that change
`const newState = {
    ...yourFirstCallToSetState,
    ...yourSecondCallToSetState,
    ...yourThirdCallToSetState,
};`
* the argument of this.setState() can be a function
    * the count after the increment call is 3
    * when a function is passed to this.setState(), it plays through each of them
`increment = () => {
    this.setState(({ count }) => { return { count: count + 1 }; });
    this.setState(({ count }) => { return { count: count + 1 }; });
    this.setState(({ count }) => { return { count: count + 1 }; });
};`
    * with props argument in the function
`increment = () => {
    this.setState((state, props) => {
        const { max, step } = props;
        if (state.count >= max) return;
        return { count: state.count + step };
    });
};`
* this.setState() can have a second argument as callback function, which is called after the state has been updated
`// max = 15, step = 5
increment = () => {
    this.setState(
        (state, props) => {
            const { max, step } = props;
            if (state.count >= max) return;
                return { count: state.count + step };
            },
        () => {
            console.log(this.state.count); // 5
        },
    );
};`
* State management patterns and anti-patterns
    * Don’t use this.state for derivations of props.
`// Don't do this
class User extends Component {
    constructor(props) {
        super(props);
        this.state = {
            fullName: props.firstName + ' ' + props.lastName,
        };
    }
}
// Instead, drive computed properties directly fro the props themselves.
class User extends Component {
    render() {
        const { firstName, lastName } = this.props;
        const fullName = firstName + ' ' + lastName;
        return <h1>{fullName}</h1>;
    }
}
// Alternatively...
class User extends Component {
    get fullName() {
        const { firstName, lastName } = this.props;
        return firstName + ' ' + lastName;
    }
    render() {
        return <h1>{this.fullName}</h1>;
    }
}`
    * Don’t use state for things you are not going to render.
`// Don't put tweetChecker into state
class TweetStream extends Component {
    constructor() {
        super();
        this.state = {
            tweet: [],
            tweetChecker: setInterval(() => {
                Api.getAll('/api/tweets').then((newTweets) => {
                    const { tweets } = this.state;
                    this.state({ tweets: [...tweets, newTweets] });
                });
            }, 1000),
        };
    }

    componentWillUnmount() {
        clearInterval(this.state.tweetChecker);
    }

    render() {}
}
// Move tweetChecker to componentWillMount
class TweetStream extends Component {
    constructor() {
        super();
        this.state = {
            tweet: [],
        };
    }

    componentWillMount() {
        this.tweetChecker = setInterval(() => {
            Api.getAll('/api/tweets').then((newTweets) => {
                const { tweets } = this.state;
                this.state({ tweets: [...tweets, newTweets] });
            });
        }, 1000);
    }

    componentWillUnmount() {
        clearInterval(this.tweetChecker);
    }

    render() {}
}`
    *  Use sensible defaults
`// Don't do this
class Item extends Component {
    constructor() {
        super();
    }

    componentDidMount() {
        Api.getAll('/api/items').then((items) => {
            this.state({ items });
        });
    }

    render() {}
}

// Always default the state
class Item extends Component {
    constructor() {
        super();
        this.state = {
            items: [],
        };
    }

    componentDidMount() {
        Api.getAll('/api/items').then((items) => {
            this.state({ items });
        });        
    }

    render() {}
}`


### Hooks State
* useState()
    * set function in useState() is asynchronous, the count after the increment call is 1
`const [count, setCount] = useState(0);
const increment = () => {
    setCount(count + 1);
    setCount(count + 1);
    setCount(count + 1);
    console.log(count); // async, count = 0
};`
    * the argument of set function in useState() can be a function
        * the count after the increment call is 3
        * the set function doesn’t have props as second argument, this is different from setState() 
`const increment = () => {
    setCount((count) => count + 1);
    setCount((count) => count + 1);
    setCount((count) => count + 1);
};`
        * always return the value, this is different from setState()
`const increment = () => {
    setCount((count) => {
        if (count >= max) return count; // always return count
        return count + step;
    });
};`
* useEffect()
    * dependencies
`// dependency: invoke the callback every time the component re-renders
useEffect(() => {
    document.title = `Counter: ${count}`;
}, []);
// dependency: only invoke the callback once
useEffect(() => {
    document.title = `Counter: ${count}`;
}, []);
// dependency: only invoke the callback if count has been updated
useEffect(() => {
    document.title = `Counter: ${count}`;
}, [count]);`
    * differences with componentDidUpdate
        * componentDidUpdate reference to the same state and props
        * useEffect gets a copy of the state and props
`componentDidUpdate() {
    setTimeout(() => {
        console.log(this.state.count); // if click increment 5 times, it will log `5 5 5 5 5`
    }, 1000);
}

useEffect(() => {
    setTimeout(() => {
        console.log(count); // if click increment 5 times, it will log `1 2 3 4 5`
    }, 1000);
}, [count]);`
    * cleanup
`useEffect(() => {
    const countTimeout = setTimeout(() => {
        console.log(count);
    }, 1000);

    return () => clearTimeout(countTimeout);
}, [count]);`
* useRef(): ability to reference to previous value
`let message;
if (cuntRef.current < count) message = 'Higher';
if (cuntRef.current > count) message = 'Lower';
cuntRef.current = count;`


### Reducers
* useReducer(): it allows you to create interfaces where you can pass in the mechanics about how to update state
`const GRUDGE_ADD = 'GRUDGE_ADD';
const GRUDGE_FORGIVE = 'GRUDGE_FORGIVE';

const reducer = (state, action) => {
    switch (action.type) {
        case GRUDGE_ADD:
            return [action.payload, ...state];
        case GRUDGE_FORGIVE:
            return state.map((grudge) => {
                if (grudge.id !== action.payload.id) return grudge;
                    return { ...grudge, forgiven: !grudge.forgiven };
                });
        default:
            return state;
    }
};
const Application = () => {
    const [grudges, dispatch] = useReducer(reducer, initialState);

    const addGrudge = ({ person, reason }) => {
        dispatch({
            type: GRUDGE_ADD,
            payload: {
                person,
                reason,
                forgiven: false,
                id: id()
            }
        });
    };

    const toggleForgiveness = (id) => {
        dispatch({
            type: GRUDGE_FORGIVE,
            payload: { id }
        });
    };
};`
* memo(): takes a function component and return one that if it receives the same props, just don’t render it
`const Grudge = memo(({ grudge, onForgive }) => {});
const NewGrudge = memo(({ onSubmit }) => {});`
* useCallback(): returns its function when the dependencies change
    * useCallback(fn, deps)
`const addGrudge = useCallback(
    ({ person, reason }) => {
        dispatch({
            type: GRUDGE_ADD,
            payload: {
            person,
            reason,
            forgiven: false,
            id: id()
            }
        });
    },
    [dispatch]
);`
* useMemo(): calls its function and returns the result when the dependencies change
    * useMemo(() => fn, deps)
`const toggleForgiveness = useMemo(
    () => (id) => {
        dispatch({
            type: GRUDGE_FORGIVE,
            payload: { id }
        });
    },
    [dispatch]
);`


Context
* Context provides a way to pass data through the component tree without having to pass props down manually at every level

`export const GrudgeContext = createContext();

export const GrudgeProvider = ({ children }) => {
    const [grudges, dispatch] = useReducer(reducer, initialState);

    const addGrudge = useCallback(
        ({ person, reason }) => {
            dispatch({
                type: GRUDGE_ADD,
                payload: {
                    person,
                    reason,
                    forgiven: false,
                    id: id()
                }        
            });
        },
        [dispatch]
    );

    const toggleForgiveness = useMemo(
        () => (id) => {
            dispatch({
                type: GRUDGE_FORGIVE,
                payload: { id }
            });
        },
        [dispatch]
    );

    const value = { addGrudge, toggleForgiveness, grudges };

    return (
        <GrudgeContext.Provider value={value}>{children}</GrudgeContext.Provider>
    );
};`
* trade-off: maintainability vs performance
    * We lost all of our performance optimization when moving to the Context API



`Data Fetching
const initialState = {
    result: null,
    loading: true,
    error: null,
};

const fetchReducer = (state, action) => {
    switch (action.type) {    
        case 'LOADING':
            return {
                result: null,
                loading: true,
                error: null,
            };
        case 'RESPONSE_COMPLETE':
            return {
                result: action.payload.response,
                loading: false,
                error: null,
            };
        case 'ERROR':
            return {
                result: null,
                loading: false,
                error: action.payload.error,
            };
        default:
            return state;
    }
};

const useFetch = (url) => {
    const [state, dispatch] = useReducer(fetchReducer, initialState);

    useEffect(() => {
        dispatch({ type: 'LOADING' });

        const fetchUrl = async () => {
            try {
                const response = await fetch(url);
                const data = await response.json();
                dispatch({ type: 'RESPONSE_COMPLETE', payload: { response: data } });
            } catch (error) {
                dispatch({ type: 'ERROR', payload: { error } });
            }
        };
        fetchUrl();
    }, [url]);

    return [state.result, state.loading, state.error];
};

const Application = () => {
    const [response, loading, error] = useFetch(endpoint + '/characters');
    const characters = (response && response.characters) || [];
    return (
        <div className="Application">        
            <header>
                <h1>Star Wars Characters</h1>
            </header>    
            <main>
                <section className="sidebar">
                    {loading ? (
                        <p>Loading...</p>
                    ) : (
                        <CharacterList characters={characters} />
                    )}
                    {error && <p className="error">{error.message}</p>}
                </section>
            </main>
        </div>
    );
};

const rootElement = document.getElementById('root');

ReactDOM.render(
    <Router>
        <Application />
    </Router>,
    rootElement,
);`

Thunks
* thunk: a function returned from another function
    * Why we need it? The major idea behind a thunk is that it’s code to be executed later, but reducers only accept objects as actions.
`function definitelyNotAThunk() {
    return function aThunk() {
        console.log('Hello, I am a thunk.')
    }
}`
* example
`const initialState = {
    characters: [],
    loading: true,
    error: null,
};

const fetchReducer = (state, action) => {
    switch (action.type) {
        case 'LOADING':
            return {
                characters: [],
                loading: true,
                error: null,
            };
        case 'RESPONSE_COMPLETE':
            return {
                characters: action.payload.characters,
                loading: false,
                error: null,
            };
        case 'ERROR':
            return {
                characters: [],
                loading: false,
                error: action.payload.error,
            };
        default:
            return state;
    }
};

const useThunkReducer = (reducer, initialState) => {
    const [state, dispatch] = useReducer(reducer, initialState);
    const enhancedDispatch = useCallback(
        (action) => {
            if (isFunction(action)) {
                action(dispatch);
            } else {
                dispatch(action);
            }
        },
        [dispatch],
    );
    return [state, enhancedDispatch];
};

const fetchCharacters = (dispatch) => {
    dispatch({ type: 'LOADING' });

    fetch(endpoint + '/characters')
        .then((response) => response.json())
        .then((response) =>
            dispatch({
                type: 'RESPONSE_COMPLETE',
                payload: { characters: response.characters },
            }),
        )
        .catch((error) => dispatch({ type: 'ERROR', payload: { error } }));
};

const Application = () => {
    const [state, dispatch] = useThunkReducer(fetchReducer, initialState);
    const { characters } = state;

    return (
        <div className="Application">
            <header>
                <h1>Star Wars Characters</h1>
            </header>
            <main>
                <section className="sidebar">
                    <button onClick={() => dispatch(fetchCharacters)}>
                        Fetch Characters
                    </button>
                    <CharacterList characters={characters} />
                </section>
                <section className="CharacterView">
                    <Route path="/characters/:id" component={CharacterView} />
                </section>
            </main>
        </div>
    );
};`
