# Frontend HTML CSS JS Calculator

Frontend Project Layout will look like this:
```
~/environment/calculator-frontend
├── aws-cli/
│   ├── artifacts-bucket-policy.json
│   ├── code-pipeline.json
│   ├── eks-calculator-codebuild-codepipeline-iam-role.yaml
│   ├── website-bucket-policy.json
│   └── website-cloudfront-distribution.json
├── base.css
├── index.html
├── querycalc.js
└── README.md
```

## Step 1: Create Frontend using HTML, CSS, JS
- Calculator triggered by end users through a web page
- TODO: Use Bootstrap for UI

### Step 1.1: Create a CodeCommit Repository
```
$ aws codecommit create-repository --repository-name calculator-frontend
```

### Step 1.2: Clone the repository
```
$ cd ~/environment
$ git clone https://git-codecommit.us-east-1.amazonaws.com/v1/repos/calculator-frontend
```

### Step 1.3: Test access to repo by adding README.md file and push to remote repository
```
$ cd ~/environment/calculator-frontend
$ echo "calculator-frontend" >> README.md
$ git add .
$ git commit -m "Adding README.md"
$ git push origin master
```

### Step 1.4: Navigate to working directory
```
$ cd ~/environment/calculator-frontend
```

### Step 1.5: Create index.html file
```
$ vi index.html
```
```
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN"
"http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">

<html xmlns="http://www.w3.org/1999/xhtml">
    <head>
        <title>Simple RESTful Calculator - Web Client</title>
    </head>
    <body>
        <div class="outer-container">
            <h1>Simple RESTful Calculator</h1>
            <form id="calcform" method="POST">
            <div class="arguments">
                Argument 1: <input type="text" id="argument1" value = ""/><br/>
                Argument 2: <input type="text" id="argument2" value = ""/><br/>
            </div>
            <div class="buttons">
                <div class="button-row">
                    <input class= "func-btn" type="button" id="add" value="Add"/>
                    <input class= "func-btn" type="button" id="subtract" value="Subtract"/>
                </div>
                <div class="button-row">
                    <input class= "func-btn" type="button" id="multiply" value="Multiply"/>
                    <input class= "func-btn" type="button" id="divide" value="Divide"/>
                </div>
                <div class="button-row">
                    <input class= "func-btn" type="button" id="sqrt" value="SquareRoot"/>
                    <input class= "func-btn" type="button" id="cbrt" value="CubeRoot"/>
                </div>		
                <div class="button-row">
                    <input class= "func-btn" type="button" id="exp" value="Exponent"/>
                    <input class= "func-btn" type="button" id="factorial" value="Factorial"/>		    
                </div>
            </div>
            </form>
        </div>

        <link rel="stylesheet" type="text/css" href="./base.css">
        <script src="http://ajax.googleapis.com/ajax/libs/jquery/1.11.1/jquery.min.js"></script>
        <script src="./querycalc.js"></script>
        <script type="text/javascript">
            // once the DOM is ready start the calculator listener function
            $( document ).ready( calcListener ); 
        </script>

    </body>
</html>
```

### Step 1.6: Create base.css file
```
$ vi base.css
```
```
body {
    margin: 0;
    padding: 0;
    background-color: white;
}

.outer-container {
    width: 500px;
    height: 500px;
    margin-left: auto;
    margin-right: auto;
    overflow: auto;
    text-align: center;
    background-color: lightblue;
}

.arguments {
    margin: auto;
    padding: 5px;
}

.buttons {
    padding: 5px;
}

.button-row {
    height: 50px;
}

.func-btn {
	-webkit-appearance: none;
    width: 50%;
    height: 50px;
    float: left;
    padding: 5px;
}
```

### Step 1.7: Create querycalc.js file
```
$ vi querycalc.js
```
```
function calcListener ( jQuery ) {
    console.log( "READY!" );

    $( "#add" ).click( function ( e ) {
        var arg1 = $( "#argument1" ).val();
        var arg2 = $( "#argument2" ).val();
        doMath (arg1, arg2, 'add');
        e.preventDefault();
    });

    $( "#subtract" ).click( function ( e ) {
        var arg1 = $( "#argument1" ).val();
        var arg2 = $( "#argument2" ).val();
        doMath (arg1, arg2, 'subtract');
        e.preventDefault();
    });

    $( "#multiply" ).click( function ( e ) {
        var arg1 = $( "#argument1" ).val();
        var arg2 = $( "#argument2" ).val();
        doMath (arg1, arg2, 'multiply');
        e.preventDefault();
    });

    $( "#divide" ).click( function ( e ) {
        var arg1 = $( "#argument1" ).val();
        var arg2 = $( "#argument2" ).val();
        doMath (arg1, arg2, 'divide');
        e.preventDefault();
    });
    
    $( "#sqrt" ).click( function ( e ) {
        var arg1 = $( "#argument1" ).val();
        doMath (arg1, 1, 'sqrt');
        e.preventDefault();
    });
    
    $( "#cbrt" ).click( function ( e ) {
        var arg1 = $( "#argument1" ).val();
        doMath (arg1, 1, 'cbrt');
        e.preventDefault();
    });       
    
    $( "#exp" ).click( function ( e ) {
        var arg1 = $( "#argument1" ).val();
        var arg2 = $( "#argument2" ).val();
        doMath (arg1, arg2, 'exp');
        e.preventDefault();
    });
    
    $( "#factorial" ).click( function ( e ) {
        var arg1 = $( "#argument1" ).val();
        doMath (arg1, 1, 'factorial');
        e.preventDefault();
    })    
    
    function doMath( arg1, arg2, resource ) {
        var textStatus, jqXHR, errorThrown = '';

        console.log( "Calling " + resource + " on " + arg1 + " and " + arg2 );
    
        try {
            arg1 = Number( arg1 );
            arg2 = Number( arg2 );
	    
            //TODO handle non-numeric inputs
            if ( isNaN(arg1) || isNaN(arg2) ) throw "NaN";

            // Flask requires a JSON string and the following content-type
            $.ajaxSetup({
                contentType: "application/json"
            });

            // Makes an ajax call to url "resource" supplying arg1 and arg2
            $.ajax({
                type: "POST",
                url: "http://localhost:5000/" + resource,
                data: JSON.stringify({ argument1: arg1, argument2: arg2 }),
                dataType: "json",
                success: function ( data ) {
                    var answer = String(data[ 'answer' ]);
                    console.log( "We got an answer! " + answer );

                    // Put the answer in argument1 and blank out argument2
                    $( "#argument1" ).val( answer );
                    $( "#argument2" ).val( '' );
                },
                error: function ( textStatus, jqXHR, errorThrown ) {
                    console.log("Something has gone wrong:" + errorThrown);
                    $( "#argument1" ).val( '' );
                    $( "#argument2" ).val( '' );
                }
            });
        }

        catch( err ) {
            console.log("Error: " + err);
            $( "#argument1" ).val( '' );
            $( "#argument2" ).val( '' );
        }
    }
}
```

### Step 1.8: Test Locally

### Step 1.9: Save changes to remote git repository
```
$ git add .
$ git commit -m "Initial"
$ git push origin master
```

### (Optional) Clean up
```
$ aws codecommit delete-repository --repository-name calculator-frontend
$ Manually delete SSH Key
$ rm ~/environment/calculator-frontend/querycalc.js
$ rm ~/environment/calculator-frontend/base.css
$ rm ~/environment/calculator-frontend/index.html
$ rm -rf ~/environment/calculator-frontend
```
