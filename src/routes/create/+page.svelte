<script>
    import PouchDB from 'pouchdb';
    let name = ''
    let calories = ''
    let weight = ''
    let user = 'antonis';
    let errors = '';
    let init = false;

    const docExists = function(){
        var db = new PouchDB(user)
        return db.get('foodtypes').then(function () {
            return Promise.resolve(true);
        }).catch(function () {
            return Promise.resolve(false);
        });
    }

    function logErrors() {
        if (name == '') {
            errors += "Name must not be empty.\n"
        }
        if (calories == '') {
            errors += "Calories must have a value.\n"
        }
        if (weight == '') {
            errors += "Weight must have a value.\n"
        }
        if (parseInt(calories) < 0) {
            errors += "Calories must not be negative.\n"
        }
        if (parseInt(weight) < 0) {
            errors += "Weight must not be negative.\n"
        }
    }

    function addFoodToDB() {
        logErrors();

        if (errors != '') {
            return;
        }

        var db = new PouchDB(user)
        var doc = {
            "_id": "foodtypes",
            "types" : [
                {
                    "name" : name,
                    "calories" : calories,
                    "weight" : weight
                }
            ]
        }
        docExists().then(function (res) {
        if (res) {
            db.get('foodtypes').then(function (olddoc) {
                var oldtypes = olddoc.types;
                var newdoc = {
                    "_id": "foodtypes",
                    "types" : [
                        {
                            "name" : name,
                            "calories" : calories,
                            "weight" : weight
                        }
                    ],
                    "_rev" : olddoc._rev
                };

                for (let i = 0; i < oldtypes.length; i++) {
                    newdoc.types.push(oldtypes[i]);
                }

                return db.put(newdoc);
            })
        } else {
            db.put(doc)
        }
        });
        init = true;
    }
</script>
<main>
    <h1>Create a food type</h1><br>
    <h2>Food name: </h2>
    <input bind:value={name} placeholder="name"/>
    <h2>Calories (in kcal) :</h2>
    <input onkeypress="return /[0-9]/i.test(event.key)" bind:value={calories} placeholder="calories"/>
    <h2>Weight (in grams) for the calories above :</h2>
    <input onkeypress="return /[0-9]/i.test(event.key)" bind:value={weight} placeholder="weight"/><br><br><br><br>
    <button3 on:click={addFoodToDB}>Add food type</button3><br><br><br>
    {#if errors != ''}
        <err>Errors : </err><br><br>
        <err>{errors}</err>
        <h2 style="color: red;">Food not added, fix errors first.</h2>
    {:else if init}
        <h2 style="color: green;">Food added!</h2>
    {/if}
</main>