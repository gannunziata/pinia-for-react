# pinia-for-react

A react state manager similar to vue Pinia.

## 1. Feature

1. You can modify the state directly outside the component (solving the problem that most current state libraries cannot be used externally)
2. State classification management (split by module)
3. Natural asynchronous support
4. Ready to use without provider root component wrapping
5. Friendly typescript prompt

## 2. Usage

```sh
npm i pinia-for-react
```

or

```sh
yarn add pinia-for-react
```

## 3. Sample Code

### 1. Global normal store

```tsx
import {defineStore} from "pinia-for-react";

const userStore = defineStore({
    state() {
        return {
            username: 'aaa',
            password: '1111',
            password1: '1111',
        }
    },
    actions: {
        setUserInfo(userInfo) {
            const state = this.$getState();
            // Can access internal methods
            this.$setState({
                ...state,
                ...userInfo
            })
        },
        // Asynchronous calls
        async syncUserInfo() {
            await new Promise(resolve => setTimeout(resolve, 2000));
            this.$patch({
                username: 'syncUserInfoaaa',
                password: 'syncUserInfo1111',
                password1: 'syncUserInfo1111',
            })
        },
        reset() {
            this.$reset()
        }
    }
});

// Satisfy the hook specification starting with use
const useUserStore = userStore;

// External update status
setInterval(() => {
    userStore.setUserInfo({username: 'I am state that is changed from outside the component', username1: new Date().getTime()})
}, 3000)

const ComponentA = () => {
    // Directly call the return value
    const [{username}] = useUserStore();
    return (
        <div>
            {username}
            <div>
                ComponentA
            </div>
        </div>
    )
}

const ComponentB = () => {
    // You can also use the useStore method hook directly
    const [userInfo, actions] = userStore.useStore();
    return (
        <div>
            {JSON.stringify(userInfo)}
            <button onClick={() => actions.setUserInfo({username: new Date().getTime()})}>
                ComponentB
            </button>
        </div>
    )
}

const ComponentC = () => {
    const [userInfo, actions] = userStore.useStore();

    return (
        <div>
            {JSON.stringify(userInfo)}
            <button onClick={() => actions.setUserInfo({username: new Date().getTime()})}>
                ComponentB
            </button>
        </div>
    )
}


export default () => {
    return (
        <div>
            <span>Hello world!</span>
            <ComponentA/>
            <ComponentB/>
            <ComponentC/>
            {/* You can also directly call the method of the returned userStore variable */}
            <button onClick={() => userStore.$reset()}>
               Status reset
            </button>
            {/* You can also directly call the method of the returned userStore variable */}
            <button onClick={() => userStore.$patch({username: '$patch username'})}>
                $patch Changing local state
            </button>
        </div>
    )
}

```

### 2. Global proxy store

```tsx
import {defineProxyStore} from "pinia-for-react";

const useUserProxyStore = defineProxyStore({
    state() {
        return {
            username: 'aaa',
            password: '1111',
            password1: '1111',
        }
    },
    actions: {
        setUserInfo(userInfo) {
            Object.assign(this.state,userInfo)
        },
        async syncUserInfo (){
            await new Promise(resolve=>setTimeout(resolve,2000));
            Object.assign(this.state,{
                username: 'syncUserInfoaaa',
                password: 'syncUserInfo1111',
                password1: 'syncUserInfo1111',
            })
        },
        reset() {
            this.$reset()
        }
    }
});
// External calls
useUserProxyStore.state.username = 'xxxxxx'

export default useUserProxyStore

```

```tsx
import React from 'react'
import useUserProxyStore from '/@/stores/user-proxy'
import {useNavigate} from "react-router-dom";
import {Button, Card, Space} from 'antd'
import './index.css';

const ComponentA = () => {
    // After deconstruction, the set value is invalid, but the latest store can still be obtained
    const [{username}, {setUserInfo}] = useUserProxyStore();
    console.log('Refresh ComponentA')
    return (
        <Card>
            <div>
                {username}
            </div>
            <Button onClick={() => setUserInfo({Component: 'ComponentA', username: new Date().getTime()})}>
                ComponentA modifies the state
            </Button>
        </Card>
    )
}

const ComponentB = () => {
    const [userInfo, {setUserInfo}] = useUserProxyStore.useStore();
    console.log('Refresh ComponentB')
    return (
        <Card>
            <div>
                {JSON.stringify(userInfo)}
            </div>
            <Button onClick={() => setUserInfo({Component: 'ComponentB', username: new Date().getTime()})}>
                ComponentB
            </Button>
        </Card>
    )
}

let count = 0;

const ComponentC = () => {
    const [userInfo, actions] = useUserProxyStore.useStore();
    console.log('Refresh ComponentC '+(++count))
    return (
        <Card>
            <div>
                {JSON.stringify(userInfo)}
            </div>
            <Button onClick={() => actions.setUserInfo({Component: 'ComponentC', username: new Date().getTime()})}>
                ComponentC
            </Button>
        </Card>
    )
}

export default () => {

    const navigate = useNavigate();

    const [state, actions] = useUserProxyStore.useStore();

    const continuityUpdate = async ()=>{
        for (let i = 0; i < 100; i++) {
            await new Promise((resolve)=>setTimeout(resolve,50))
            useUserProxyStore.$patch({
                username: Math.random()+'',
            })
        }
    }

    return (
        <div className='home-page'>
            <h1>Index Page</h1>
            <Space direction={'vertical'}>
                <Space>
                    <ComponentA/>
                    <ComponentB/>
                    <ComponentC/>
                </Space>
                <Space>
                    <Button onClick={() => useUserProxyStore.$reset()}>
                       Status reset
                    </Button>
                    <Button onClick={continuityUpdate}>
                        Update status 10 times in a row
                    </Button>
                    <Button onClick={() => state.username = new Date().getTime()+''}>
                        $patch Changing local state
                    </Button>
                    <Button onClick={() => useUserProxyStore.syncUserInfo()}>
                        Changing state asynchronously
                    </Button>
                    <Button type="primary" onClick={() => navigate('/home')}>Jump to Home page</Button>
                </Space>
            </Space>
        </div>
    )
}

```

## 4. Running the Example

#### 1. Install Dependencies

`yarn & npm run install`

#### 2. Run web or taro

`yarn run start:web` 

or

`yarn run start:taro` 

