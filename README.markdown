# form2js
_Convenient way to collect form data into JavaScript object._

Example: http://form2js.googlecode.com/hg/example/test.html

If you have any questions/suggestions or find out something weird or illogical - feel free to post an issue.

Because everythins is better with jQuery, jQuery plugin added, check out jquery.toObject.js =)

## Details

Structure of resulting object defined by "name" attribute of form fields. See examples below.

This is **not** a serialization library. Library used in example for JSON serialization is http://www.json.org/js.html 

All this library doing is collecting form data and putting it in javascript object (obviously you can get JSON/XML/etc string by serializing it, but it's not an only purpose).

## Usage
    form2object(rootNode, delimiter, skipEmpty, nodeCallback)

Values of all inputs under the _rootNode_ will be collected into one object (skipping empty inputs if _skipEmpty_ not false).

### Objects/nested objects
Structure of resulting object defined in "name" attribute, _delimiter_ is "." (dot) by default, but can be changed.

    <input type="text" name="person.name.first" value="John" />
    <input type="text" name="person.name.last" value="Doe" />

becomes

    {
        "person" :
        {
            "name" :
            {
                "first" : "John",
                "last" : "Doe"
            }
        }
    }

### Arrays
Several fields with the same name with brackets defines array of values.

    <label><input type="checkbox" name="person.favFood[]" value="steak" checked="checked" /> Steak</label>
    <label><input type="checkbox" name="person.favFood[]" value="pizza"/> Pizza</label>
    <label><input type="checkbox" name="person.favFood[]" value="chicken" checked="checked" /> Chicken</label>

becomes

    {
        "person" :
        {
            "favFood" : [ "steak", "chicken" ]
        }
    }

### Arrays of objects/nested objects
Same index means same item in resulting array. Index doesn't specify order (order of appearance in document will be used).

    <dl>
        <dt>Give us your five friends' names and emails</dt>
        <dd>
            <label>Email <input type="text" name="person.friends[0].email" value="agent.smith@example.com" /></label>
            <label>Name <input type="text" name="person.friends[0].name" value="Smith Agent"/></label>
        </dd>
        <dd>
            <label>Email <input type="text" name="person.friends[1].email" value="n3o@example.com" /></label>
            <label>Name <input type="text" name="person.friends[1].name" value="Thomas A. Anderson" /></label>
        </dd>
    </dl>

becomes

    {
        "person" :
        {
            "friends" : [
                { "email" : "agent.smith@example.com", "name" : "Smith Agent" },
                { "email" : "n3o@example.com", "name" : "Thomas A. Anderson" }
            ]
        }
    }

## Why not to use jQuery .serializeArray() or similar functions in other frameworks instead?
.serializeArray() works a bit different. It makes this structure from markup in previous example:

    [
        { "person.friends[0].email" : "agent.smith@example.com" },
        { "person.friends[0].name" : "Smith Agent" },
        { "person.friends[1].email" : "n3o@example.com" },
        { "person.friends[1].name" : "Thomas A. Anderson" }
    ]

### Custom fields
You can implement custom nodeCallback function (passed as 4th parameter to form2object()) to extract custom data:

    <dl id="dateTest">
		<dt>Date of birth:</dt>
		<dd data-name="person.dateOfBirth" class="datefield">
			<select name="person.dateOfBirth.month">
				<option value="01">January</option>
				<option value="02">February</option>
				<option value="03">March</option>
				<option value="04">April</option>
				<option value="05">May</option>
				<option value="06">June</option>
				<option value="07">July</option>
				<option value="08">August</option>
				<option value="09">September</option>
				<option value="10">October</option>
				<option value="11">November</option>
				<option value="12">December</option>
			</select>
			<input type="text" name="person.dateOfBirth.day" value="1" />
			<input type="text" name="person.dateOfBirth.year" value="2011" />
		</dd>
	</dl>

	<script type="text/javascript">
    	function processDate(node)
    	{
    		var dataName = node.getAttribute ? node.getAttribute('data-name') : '',
    		    dayNode,
    		    monthNode,
    		    yearNode,
    		    day,
    		    year,
    		    month;

    		if (dataName && dataName != '' && node.className == 'datefield')
    		{
    			dayNode = node.querySelector('input[name="'+dataName + '.day"]');
    			monthNode = node.querySelector('select[name="'+dataName + '.month"]');
    			yearNode = node.querySelector('input[name="'+dataName + '.year"]');

    			day = dayNode.value;
    			year = yearNode.value;
    			month = monthNode.value;

    			return { name: dataName, value:  year + '-' + month + '-' + day};
    		}

    		return false;
    	}

    	var formData = form2object('dateTest', '.', true, processDate);
    </script>

using _processDate()_ callback _formData_ will contain

    {
    	"person": {
    		"dateOfBirth": "2011-01-12"
    	}
    }
