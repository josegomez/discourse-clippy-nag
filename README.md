# discourse-clippy-nag
For our own discourse site we implemented a nagging clippy which reminds users to format their code. This is how we did it.

# Setup
* Edit your Themes CSS/HTMl to add all the required CSS and Java Script below
* Clippy will fire if the tag named "codeformat" is added to a post and the current person logged in is the author of the topic (first post)

# HEAD Section Edits
```javascript
//Bring in Clippy from the latest published package
<script src="https://unpkg.com/clippyjs_html@0.0.15"></script>
<link rel="stylesheet" type="text/css" href="https://gitcdn.xyz/repo/pi0/clippyjs/master/assets/clippy.css">
<script>
//Array of all possible clippy gestures
var gestures = ["CheckingSomething",
"Congratulate",
"EmptyTrash",
"Explain",
"GestureDown",
"GestureLeft",
"GestureRight",
"GestureUp",
"GetArtsy",
"GetAttention",
"GetTechy",
"GetWizardy",
"GoodBye",
"Greeting",
"Hearing_1",
"Hide",
"Idle1_1",
"IdleAtom",
"IdleEyeBrowRaise",
"IdleFingerTap",
"IdleHeadScratch",
"IdleRopePile",
"IdleSideToSide",
"IdleSnooze",
"LookDown",
"LookDownLeft",
"LookDownRight",
"LookLeft",
"LookRight",
"LookUp",
"LookUpLeft",
"LookUpRight",
"Print",
"Processing",
"RestPose",
"Save",
"Searching",
"SendMail",
"Show",
"Thinking",
"Wave",
"Writing"];
//Global variable to stop clippy
var stopall=false;
//little function which makes clippy do a random gesture from the above list every 4 seconds
function clipAway(agent)
{
    if(!stopall)
    {
    var item = gestures[Math.floor(Math.random()*gestures.length)];
    agent.play(item);
    setTimeout(function() {
        clipAway(agent);
    }, 4000);
    }
    stopall=false;
}
//List of instructions to nag user with
var directions = ["You got called out by your fellow community members for not formatting the code in this post",
                "as the format police, I've been sent here to ensure you know how to properly do this",
                "it is very simple, simply put 3 back ticks before and after the code",
                "```<br/> Console.Write(\"My Code here\");<br/> ```",
                "seriously! it is that simple! Now get to it!",
                "by the way to dismiss me, remove the codeformat tag from this post and I won't be here next time ",
                "for now you are stuck with me, unless you navigate away and refresh, for now we are BFFs 😘",
                "more help on this topic can be found <a href=\"https://e10help.com/t/code-syntax-highlighting-in-posts\">here</a>"];
                
//Function to make clippy speak instructions                
function agentInstructions(agent, idx)
{
    if(!stopall)
    {
        if(directions.length-1==idx || idx==3)
            agent.speak(directions[idx],undefined,13000);
        else
            agent.speak(directions[idx]);
        if(idx<directions.length-1)
            setTimeout(function() {
                agentInstructions(agent,++idx);
                console.log('call');
            }, (6000));
        else
        {
            clipAway(agent);
        }
    }
    stopall=false;
}

//Fired on page / post change evaluate if the tag called codeformat was added to the post
//if the tag exists and the current user is the author of the post call clippy to action
api.onPageChange(() =>{
            //Wait 3 seconds trying to make sure that the post has finished loading... there may be a better way to do this
          setTimeout(function() {
                var u = Discourse.User.current();
                if (u !== null && !window.mobileAndTabletcheck()){
                    //Get the author of the topic
                    var author = $("#post_1").find(".first.username").children("a").attr("data-user-card");
                    //get the format tag
                	var format = $(".discourse-tags").children("a[data-tag-name='codeformat']").attr("data-tag-name");
                	
                    //If the author matches the logged in user and the format tag is define, call onto clippy to NAG
                    if(author === u.username && format!==undefined  && clp==undefined)
                    {
                            clippy.load('Clippy', function(agent){
                            clp=agent;     
                            agent.show();
                            agent.moveTo(100,100)
                            agent.play('GetAttention');
                            agent.speak('Hello ' + author + ' good to see you are back... So this is awkward...');
                            setTimeout(function() {
                            agentInstructions(agent,0);
                            },6000);

                        });
                    }
                    //Otherwise stop nagging
                    else if(clp!=undefined && format==undefined)
                    {
                        clp.stop();
                        clp.hide(false,null);
                        clp=undefined;
                        stopall=true;
                    }
                	    
                }
          },3000);
    });
```
