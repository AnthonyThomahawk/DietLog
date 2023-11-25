<script lang="ts">
    import PouchDB from 'pouchdb';
    let name = '';
    let time = '';
    let date = '';
    let errors = '';
    let init = false;
    let user = 'antonis';
    
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

    function addMeal() {
        var db = new PouchDB(user);
        db.info().then(function (info) {
            console.log(info);
        })
        db.get('foodtypes').then(function (doc) {
            console.log(doc);
        });

    }
</script>

<main>
    <h1>Add a meal</h1>
    <h2>Name :</h2>
    <input bind:value={name} placeholder="name"/>
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
        <h2 style="color: green;">Meal added!</h2>
    {/if}
</main>