<script lang="ts">
    import PouchDB from 'pouchdb';
    import {onMount} from "svelte";
    let name = '';
    let time = '';
    let date = '';
    let errors = '';
    let init = false;
    let user = 'antonis';

    interface food {
        index : string;
        name : string;
        weight : number;
        calories : number;
    }

    let allFood : Array<food> = [];
    let allFoodChanges = 0;

    let foodInMeal : Array<food> = [];
    let foodInMealChanges = 0;
    
    function isValidDate() {
        if(!/^\d{1,2}\/\d{1,2}\/\d{4}$/.test(date))
            return false;

        var parts = date.split("/");
        var day = parseInt(parts[1], 10);
        var month = parseInt(parts[0], 10);
        var year = parseInt(parts[2], 10);

        if(year < 1000 || year > 3000 || month == 0 || month > 12)
            return false;

        var monthLength = [ 31, 28, 31, 30, 31, 30, 31, 31, 30, 31, 30, 31 ];

        if(year % 400 == 0 || (year % 100 != 0 && year % 4 == 0))
            monthLength[1] = 29;

        return day > 0 && day <= monthLength[month - 1];
    }

    function logErrors() {
        errors = '';
        init = true;
        if (name == '') {
            errors += "Name cannot be empty.\n"
        }
        if (time == '') {
            errors += "Time cannot be empty.\n"
        }
        if (date == '') {
            errors += "Date cannot be empty.\n"
        }
        if (foodInMeal.length == 0) {
            errors += "Meal must contain atleast 1 food.\n"
        }
        var isTimeValid = (time.search(/^\d{2}:\d{2}$/) != -1) &&
            (parseInt(time.substr(0,2)) >= 0 && parseInt(time.substr(0,2)) <= 24) &&
            (parseInt(time.substr(3,2)) >= 0 && parseInt(time.substr(3,2)) <= 59)
        
        if (!isTimeValid) {
            errors += "Invalid time given (It must be in 24-hour format HH:MM)\n"
        }

        if (!isValidDate()) {
            errors += "Invalid date given (It must have format MM/DD/YYYY)\n"
        }
    }

    const docExists = function(){
        var db = new PouchDB(user)
        return db.get('meals').then(function () {
            return Promise.resolve(true);
        }).catch(function () {
            return Promise.resolve(false);
        });
    }

    onMount(async () => {
        getFoodTypes();
    });

    async function getFoodTypes() {
        var db = new PouchDB(user);
        db.get('foodtypes').then(function (doc) {
            let foodarr = doc.types;

            for (let i = 0; i < foodarr.length; i++) {
                const f : food = {
                    index : i.toString(),
                    name : foodarr[i].name,
                    weight : foodarr[i].weight,
                    calories : foodarr[i].calories
                }
                allFood.push(f);
            }
            allFoodChanges++;
        });
    }

    function addToMeal(index : string) {
        let i = parseInt(index);
        foodInMeal.push(allFood[i]);
        foodInMealChanges++;
    }

    function remFromMeal(index : string) {
        let i = parseInt(index);
        foodInMeal.splice(i, 1);
        for (let j = 0; j < foodInMeal.length; j++) {
            foodInMeal[j].index = j.toString();
        }
        foodInMealChanges++;
    }

    function addMeal() {
        logErrors();
        if (errors == '') {
                var db = new PouchDB(user);
            db.info().then(function (info) {
                console.log(info);
            })
            db.get('foodtypes').then(function (doc) {
                console.log(doc);
            });
        }
    }
</script>

<main>
    <h1>Add a meal</h1>

    <h2>Name :</h2>
    <input bind:value={name} placeholder="name"/>
    <h2>Available foods to add : </h2>
    <div class="container">
        <table>
            <thead>
            <tr>
                <th>Food</th>
                <th>Weight</th>
                <th>Calories</th>
                <th></th>
            </tr>
            </thead>
            <tbody>
                {#key allFoodChanges}
                    {#each allFood as f}
                        <tr>
                            <td>{f.name}</td>
                            <td>{f.weight} g</td>
                            <td>{f.calories} kcal</td>
                            <td><button3 on:click={() => addToMeal(`${f.index}`)}>Add to meal</button3></td>
                        </tr>
                    {/each}
                {/key}
            </tbody>
        </table>
    </div>
    <h2>Meal consists of :</h2>
    <div class="container">
        <table>
            <thead>
            <tr>
                <th>Food</th>
                <th>Weight</th>
                <th>Calories</th>
                <th></th>
            </tr>
            </thead>
            <tbody>
                {#key foodInMealChanges}
                    {#each foodInMeal as f}
                        <tr>
                            <td>{f.name}</td>
                            <td>{f.weight} g</td>
                            <td>{f.calories} kcal</td>
                            <td><button3 on:click={() => remFromMeal(`${f.index}`)}>Delete</button3></td>
                        </tr>
                    {/each}
                {/key}
            </tbody>
        </table>
    </div>
    <h2>Time eaten (24-hour format) :</h2>
    <input onkeypress="return /[0-9:]/i.test(event.key)" bind:value={time} placeholder="time"/>
    <h2>Day eaten (MM/DD/YYYY)</h2>
    <input bind:value={date} placeholder="date"/>
    <br><br><br>
    <button3 on:click={addMeal}>Add meal</button3>
    <br><br><br>
    {#if errors != ''}
        <err>Errors : </err><br><br>
        <err>{errors}</err>
        <h2 style="color: red;">Meal not added, fix errors first.</h2>
    {:else if init}
        <h2 style="color: green;">Meal "{name}" added!</h2>
    {/if}
</main>

<style>
    table {
        margin: 0 auto;
        border-collapse: collapse;
        width: auto;
        background-color: #fff;
        box-shadow: 0 0 20px rgba(0,0,0,0.15);
        table-layout: fixed;
        border: 1px solid #ddd;
        border-top: none;
        border-bottom: none;
    }

    tr {
        border-bottom: 1px solid #ddd;
    }

    th, td {
        padding: 12px 15px;
        text-align: left;
        border-bottom: 1px solid #ddd;
    }

    tr:hover {
        background-color: #f5f5f5;
    }

    th {
        background-color: #12171a;
        color: white;
        font-weight: bold;
    }

    .qty {
        width: 60px;
        height: 30px;
        padding: 0 10px;
        border: 1px solid #ccc;
        border-radius: 4px;
        border-bottom: 3px solid #ddd;
        box-sizing: border-box;
        font-size: 100%;
    }
    .container table {
        overflow-x: hidden;
    }

    td:last-child {
        border-bottom: none;
    }

    main {
        height: 100vh;
        widows: 100vw;
        overflow: auto;
        padding: 20px;
    }
</style>