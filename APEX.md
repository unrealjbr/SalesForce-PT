# Interacting with Apex Class Methods

Thus far, everything has been quite straightforward and that process will not change for any target. The ability to exploit these insecure methods will separate the wheat from the chaff. First things first, a basic understanding of how to efficiently understand these from a blackbox perspective is important.

When filtering through your Burp Proxy history, or sliding down a mass of requests, there are two simple ways to find any custom apex class definitions or calls:

    The string ‘apex%3a%2f%2f’ in the request 

    The string ‘compound://c’ in the response

I will focus on the second, as ultimately any apex call that is made in a background request will lead you to look for the actual descriptor itself anyway.

Below is an snippet of what you may come across in a response when searching for this string:

"componentDefs":[
{"xs":"G","descriptor":"markup://c:SampleComponent"
..snip..
"cd":{
      "descriptor":"compound://c.SampleComponent",
      "ac":[
        { 
           "n":"getBaseUrl",
           "descriptor":"apex://SampleController/ACTION$getBaseUrl","at":"SERVER","rt":"apex://Map<String,String>",
           "pa":[{"name":"url","type":"apex://String"}]
..snip..

The important values to note here are:

    The initial ‘discriptor’ value that exists in ‘componentDefs’ - This can be used for retrieving the full definition, although not required. The descriptor format is made up of namespace:component.

    rt - The return value type. In this case, it’s a Map.

    pa - Parameters that are passed to the apex class method, and their type. Here the parameter accepted is “url” of type “String”.

Knowing this information, we can attempt to interact with this method and see what it returns. Here’s the constructed message value:

{"actions":[{"id":"46;a","descriptor":"apex://NovaBaseController/ACTION$getBaseUrl","callingDescriptor":"UNKNOWN","params":{"url":"https://google.com/something"}}]}

Let’s break this down into points:

    id - Completely irrelevant, enter 1337 here if it makes you feel better

    descriptor - The controller and subsequent method we are calling

    callingDescriptor - Okay, so technically in an ideal world this would contain the componentDef markup string, but I have not seen it ever required so “UNKNOWN” is accepted across the board

    params - This JSON object contains the ‘url’ parameter and value that I’ve decided to give it, which is a URL. I simply looked at the parameter name and took a wild guess, welcome to hacking :)

Submitting the request returned the following value:

"returnValue":{
     "completeUrl":"https://google.com/something",
     "pathUrl":"/something"
     }

Some of you may be thinking “When searching for custom apex classes, the responses are so cluttered and it’s hard to focus. How do I see JUST the methods for a particular custom class?”.  This can be retrieved via the ‘/auraCmpDef’ endpoint. The endpoint itself requires a few pieces of information prior to accessing it, as seen below:

/auraCmpDef?aura.app=<APP_MARKUP>&_au=<AU_VALUE>&_ff=DESKTOP&_l=true&_cssvar=false&_c=false&_l10n=en_US&_style=-1450740311&_density=VIEW_ONE&_def=<COMPONENT_MARKUP>

The values for these ‘aura.app’ and ‘_au’ parameters can be found in two places. side by side. Firstly when a call to a particular method of a class is called in a request, it can be found in the ‘aura.context’ POST parameter’s value. Secondly, in the response that describes the custom class itself (CTRL+F ‘Application@’):
[Photo](load.png)

In the example request above, the ‘aura.app’ value is ‘markup://siteforce:communityApp’ and the _au value is ‘8KVdMoLuAGi15YkxlC35vw’. Lastly is the component descriptor value is required for the ‘_def’ parameter,and you may have noticed one earlier in this subsection. Search for ‘"descriptor":"markup://c’ in the response where these methods are outlined, and copy the entire value for the descriptor. 
[Photo](load2.png)
 Plugging in these values would leave us with the following finished path & parameters (Note that the ‘/auraCmpDef’ endpoint is in the same directory as ‘/aura’ is found):

/auraCmpDef?aura.app=markup://siteforce:communityApp&_au=8KVdMoLuAGi15YkxlC35vw&_ff=DESKTOP&_l=true&_cssvar=false&_c=false&_l10n=en_US&_style=-1450740311&_density=VIEW_ONE&_def=markup://c:NOVABaseComponent

Navigating to the URL will perform a 302 redirect to what we seek. Below is a snippet ‘/auraCmpDef’ output for a built-in method within a component.
Example method description from ‘/auraCmpDef’ for the built in ‘forceSearch:resultsGridLVMDataManager’ It’s bad enough that you’re often guessing parameter values for parameters of a String type, but even worse are parameters which expect an object, as you’re now dealing with potentially quite a few blind parameters within said object itself and their respective values. Here’s a fairly obvious and handy trick for this. If an object is expecting a return type of ‘aura://User’, check to see if any other apex methods have a ‘rt’ value of ‘aura://User’, and then use the output from that method as the input for the first.
[Photo](load3.png)

Example method description from ‘/auraCmpDef’ for the built in ‘forceSearch:resultsGridLVMDataManager’ 

It’s bad enough that you’re often guessing parameter values for parameters of a String type, but even worse are parameters which expect an object, as you’re now dealing with potentially quite a few blind parameters within said object itself and their respective values. Here’s a fairly obvious and handy trick for this. If an object is expecting a return type of ‘aura://User’, check to see if any other apex methods have a ‘rt’ value of ‘aura://User’, and then use the output from that method as the input for the first.
