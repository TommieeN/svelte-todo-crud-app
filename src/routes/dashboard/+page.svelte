<script>
  import Todoitem from "../../components/Todoitem.svelte";
import { db } from "../../lib/firebase/firebase"
  import { authHandlers, authStore } from "../../store/store";
  import { doc, setDoc } from "@firebase/firestore";
    let todoList = [];
    let currTodo = ""
    let error = false

    // authStore.subscribe(curr => {
    //     todoList = curr.data.todos;
    // })

    authStore.subscribe(curr => {
    if (curr.data && Array.isArray(curr.data.todos)) {
        todoList = curr.data.todos;
    } else {
        todoList = []; // Default to an empty array if the structure is unexpected
    }
})

    function addTodo() {
        error = false
        if (!currTodo) {
            error = true;
        }
         todoList = [...todoList, (currTodo)]
         currTodo = ""
    }

    function editTodo(index) {
        let newTodoList = todoList.filter((val, i) => {
            return i !== index;
        })
        currTodo = todoList[index]
        todoList = newTodoList
    }

    function removeTodo(index) {
        let newTodoList = todoList.filter((val, i) => {
            return i !== index;
        })
        todoList = newTodoList
    }

    async function saveTodos() {
        try {
            const userRef = doc(db, "users", $authStore.user.uid)
            await setDoc(
                userRef,
            {
                todos: todoList,
            },
            { merge: true }
            )
        } catch (err) {
            console.log("There was an error saving your informatioin")
        }
    }

</script>

{#if !$authStore.loading}
<div class="mainContainer">
    <div class="headerContainer">
        <h1>Todo List</h1>
        <div class="headerButtons">
        <button on:click={saveTodos} >
            <i class="fa-regular fa-floppy-disk"></i><p>Save</p>
        </button>
        <button on:click={authHandlers.logout}>
            <i class="fa-solid fa-right-from-bracket"></i><p>Logout</p>
        </button>
    </div>
    </div>
    <main>
        {#if todoList.length === 0}
        <p>
            You have nothing to do!
        </p>
        {/if}
        {#each todoList as todo, index}
            <Todoitem todo={todo} index={index} removeTodo={removeTodo} editTodo={editTodo} />
        {/each}
    </main>
    <div class={"enterTodo " + (error ? "errorBorder" : "")}>
        <input bind:value={currTodo} type="text" placeholder="Enter todo"/>
        <button on:click={addTodo}>ADD</button>
    </div>
</div>
{/if}

<style>
    .mainContainer {
        display: flex;
        flex-direction: column;
        min-height: 100vh;
        gap: 24px;
        padding: 24px;
        width: 100%;
        max-width: 1000px;
        margin: 0 auto;
    }

    .headerContainer {
        display: flex;
        align-items: center;
        justify-content: space-between;
    }

    .headerButtons {
        display: flex;
        align-items: center;
        gap: 14px;
    }

    .headerContainer button {
        background: #003c5b;
        color: white;
        padding: 10px 18px;
        border: none;
        border-radius: 4px;
        font-weight: 700;
        display: flex;
        align-items: center;
        gap: 10px;
        cursor: pointer;
    }

    .headerContainer button i {
        font-size: 1.1rem;
    }

    .headerContainer button:hover {
        opacity: 0.7;
    }

    main {
        display: flex;
        flex-direction: column;
        gap: 8px;
        flex: 1;
    }

    .todo {
        border-left: 1px solid cyan;
        padding: 8px 14px;
        display: flex;
        align-items: center;
        justify-content: space-between;
    }

    .actions {
        display: flex;
        align-items: center;
        gap: 14px;
        font-size: 1.3rem;
    }

    .errorBorder {
        border-color:  coral !important;
    }

    .enterTodo {
        display: flex;
        align-items: stretch;
        border: 1px solid #0891b2;
        border-radius: 5px;
        overflow: hidden;
    }

    .enterTodo input {
        background: transparent;
        border: none;
        padding: 14px;
        color: white;
        flex: 1;
    }

    .enterTodo input:focus {
        outline: none;
    }

    .enterTodo button {
        padding: 0 28px;
        background: #003c5b;
        border: none;
        color: cyan;
        font-weight: 600;
        cursor: pointer;
    }

    .enterTodo button:hover {
        background: transparent;
    }
</style>