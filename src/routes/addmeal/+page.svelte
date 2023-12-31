<script lang="ts">
    import PouchDB from 'pouchdb';
    import {onMount} from 'svelte';
    let name = '';
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

    let mealTime = '';
    let totalWeight = 0;
    let totalKcal = 0;

    let nofoodtypes = false;
    
    function isValidDate() {
        if(!/^\d{2}\/\d{2}\/\d{4}$/.test(date))
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
        if (date == '') {
            errors += "Date cannot be empty.\n"
        }
        if (foodInMeal.length == 0) {
            errors += "Meal must contain atleast 1 food.\n"
        }
        if (mealTime == '') {
            errors += "Meal time is not set.\n"
        }

        if (!isValidDate()) {
            errors += "Invalid date given (It must have format MM/DD/YYYY)\n"
        }
    }

    const docExists = function(doc : string){
        var db = new PouchDB(user)
        return db.get(doc).then(function () {
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

        docExists('foodtypes').then(function(res) {
            if (res) {
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
                    nofoodtypes = false;
                });
            } else {
                nofoodtypes = true;
            }
        });
    
    }

    function calcTotalWeight() {
        totalWeight = 0;
        for (let i = 0; i < foodInMeal.length; i++) {
            totalWeight += foodInMeal[i].weight;
        }
    }

    function calcTotalKcal() {
        totalKcal = 0;
        for (let i = 0; i < foodInMeal.length; i++) {
            totalKcal += foodInMeal[i].calories;
        }
    }

    function addToMeal(index : string) {
        let i = parseInt(index);
        let clone = Object.assign({}, allFood[i]);
        foodInMeal.push(clone);
        calcTotalKcal();
        calcTotalWeight();
        foodInMealChanges++;
    }

    function remFromMeal(index : string) {
        let i = parseInt(index);
        foodInMeal.splice(i, 1);
        for (let j = 0; j < foodInMeal.length; j++) {
            foodInMeal[j].index = j.toString();
        }
        calcTotalKcal();
        calcTotalWeight();
        foodInMealChanges++;
    }

    function addMeal() {
        logErrors();
        if (errors == '') {
            var db = new PouchDB(user);

            var meal = {
                "name" : name,
                "time" : mealTime,
                "date" : date,
                "foodInMeal" : []
            }

            for (let i = 0; i < foodInMeal.length; i++) {
                var f = {
                    "name" : foodInMeal[i].name,
                    "calories" : foodInMeal[i].calories,
                    "weight" : foodInMeal[i].weight
                }
            }

            meal.foodInMeal.push(f);

            docExists('meals').then(function (res) {
                    if (res) {
                        db.get('meals').then(function (olddoc) {
                            var oldmeals = olddoc.allmeals;
                            var newdoc = {
                                "_id" : "meals",
                                "allmeals" : [],
                                "_rev" : oldmeals._rev
                            };

                            for (let i = 0; i < oldmeals.length; i++) {
                                newdoc.allmeals.push(oldmeals[i]);
                            }

                            newdoc.allmeals.push(meal);
                            db.put(newdoc);
                        })
                    }
                    else {
                        var doc = {
                            "_id" : "meals",
                            "allmeals": [meal]
                        }
                        db.put(doc);
                    }
                }
            );
        }
    }

    function onChange(event) {
		mealTime = event.currentTarget.value;
	}
</script>

<main>
    <h1>Add a meal</h1>

    <h2>Name :</h2>
    <input bind:value={name} placeholder="name"/>
    {#if nofoodtypes}
        <h2>There are no available foods to add. Please add some here</h2>
    {:else}
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
    {/if}
    {#if foodInMeal.length == 0}
        <h2>Meal does not have any items. (Add items using "Add to meal" button)</h2>
    {:else}
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
            {#key foodInMealChanges}
                <h3>Total weight : {totalWeight} g</h3>
                <h3>Total calories : {totalKcal} kcal</h3>
            {/key}
        </div>
    {/if}
    <h2>Meal time :</h2>
    <label style="color: white;">
        <input checked={mealTime==="breakfast"} on:change={onChange} type="radio" name="amount" value="breakfast" /> breakfast
    </label>
    <label style="color: white;">
        <input checked={mealTime==="brunch"} on:change={onChange} type="radio" name="amount" value="brunch" /> brunch
    </label>
    <label style="color: white;">
        <input checked={mealTime==="lunch"} on:change={onChange} type="radio" name="amount" value="lunch" /> lunch
    </label>
    <label style="color: white;">
        <input checked={mealTime==="supper"} on:change={onChange} type="radio" name="amount" value="supper" /> supper
    </label>
    <label style="color: white;">
        <input checked={mealTime==="dinner"} on:change={onChange} type="radio" name="amount" value="dinner" /> dinner
    </label>
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
        width: auto;
        overflow: auto;
        padding: 20px;
    }
</style>