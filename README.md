# mailchimp-in-dnn
How to add a mailchimp for to a DNN website natively (no iframes needed)
 
Adding a form to DNN without using a module can be tough if you're not a developer. DNN wraps the content of the website inside a form tag, which means you cannot place another form inside it.
DNN modules have to be built differently for this. While you can find plenty of modules in the DNN store to add Mailchimp to your website, not everyone is willing to shell out $100+ just to include a simple form.
At BBI, we tackled the challenge of integrating a Mailchimp form into DNN and nailed it. 
The problem
As mentioned, DNN wraps the entire website in a form. The form tag for DNN is required to allow administrative tasks to happen. In order to perform tasks such as adding or removing modules, the presence of the form element is necessary.

In DNN, the form tag is the first thing you see in the source code after the body tag and looks like this:
<form method="post" action="/" id="Form" enctype="multipart/form-data">

So, if you then try to include another form on your page, it won't work as mentioned already. 

The first task is to write some code which will eliminate the form tag from the DOM. However, you need to ensure that this is only affected when you are logged out. If you do not do this, then you cannot administer your DNN website.
In your DNN skin, you need to first work out if your site is running on C# or VB as the code is slightly different for each version.
You can do this by looking at your skin file and looking at the very first line of the skin file. I our case you can see that we are running c#
<%@ Control language="C#" AutoEventWireup="false" Explicit="True" Inherits="DotNetNuke.UI.Skins.Skin" %>

If you have a VB based skin use this:
<% If Not Request.IsAuthenticated Then %>
Script goes here
<%  End If %>



If you have a C# based skin use this:

<% if (!Request.IsAuthenticated) {%>
Script goes here
<%}%>
Next we need to ensure that the form that wraps the site is replaced with a div.

Once again, it is this code we need to replace:
<form method="post" action="/newsletter" id="Form" enctype="multipart/form-data">

Within either of the 2 codes add this:

  var el2 = document.querySelector('form#Form');
        el2.outerHTML = '<div class="gone">' + el2.innerHTML + '</div>';

So if you run C#, it will look like this:

<% if (!Request.IsAuthenticated) {%>
  <script>
    var el2 = document.querySelector('form#Form');
        el2.outerHTML = '<div class="gone">' + el2.innerHTML + '</div>';
  </script>
<%}%>


Create a skin which holds your newsletter and add this to the skin, that way it only affects the page your Mailchimp form lives on.
Next, you want to edit your Mailchimp form as the form tag in your Mailchimp for will still vanish if you try to add it.


In the Mailchimp form below
Where highlighted you will see we have included an extra div

<div id="mc_embed_signup">
    <div class="iamform">
        <form
            action="https://bbi.us5.list-manage.com/subscribe/post?u=bdb648829adf3af97bbd37b0f&amp;id=6cabd3ccaf&amp;f_id=00d9c2e1f0"
            method="post" id="mc-embedded-subscribe-form" name="mc-embedded-subscribe-form" class="validate"
            target="_blank">
            <div id="mc_embed_signup_scroll">
                <h2>Subscribe</h2>
                <div class="indicates-required"><span class="asterisk">*</span> indicates required</div>
                <div class="mc-field-group"><label for="mce-FNAME">First Name </label><input type="text" name="FNAME"
                        class=" text" id="mce-FNAME" value=""></div>
                <div class="mc-field-group"><label for="mce-EMAIL">Email Address <span
                            class="asterisk">*</span></label><input type="email" name="EMAIL" class="required email"
                        id="mce-EMAIL" value="" required=""></div>
                <div id="mce-responses" class="clear">
                    <div class="response" id="mce-error-response" style="display: none;"></div>
                    <div class="response" id="mce-success-response" style="display: none;"></div>
                </div>
                <div style="position: absolute; left: -5000px;" aria-hidden="true"><input type="text"
                        name="b_bdb648829adf3af97bbd37b0f_6cabd3ccaf" tabindex="-1" value=""></div>
                <div class="clear"><input type="submit" name="subscribe" id="mc-embedded-subscribe"
                        class="button redbutton" value="Subscribe"></div>
            </div>
        </form>
    </div>
</div>

This is important because the original form div is deleted (you can see I have highlighted it in grey) from the DOM, therefore the form will not work. So we will be using the new div with the class of “iamform”  (highlighted in yellow) which will be turned into a form tag with the corresponding action.

So back in your skin where you added the code to remove the Form tag from your DNN page, add the following

var el3 = document.querySelector('.iamform');
        el3.outerHTML = '  <form action="https://bbi.us5.list-manage.com/subscribe/post?u=bdb648829adf3af97bbd37b0f&amp;id=6cabd3ccaf&amp;f_id=00d9c2e1f0" method="post" id="mc-embedded-subscribe-form" name="mc-embedded-subscribe-form" class="validate" target="_blank">' + el3.innerHTML + '</form>';

So back in your skin where you added the code to remove the Form tag from your DNN page, add the following.

Where I have highlighted it in yellow, this is your own form URL which you can get from your form code provided by Mailchimp.

In your skin file, your code will look like this
<% if (!Request.IsAuthenticated) {%>
  <script>
    var el2 = document.querySelector('form#Form');
        el2.outerHTML = '<div class="gone">' + el2.innerHTML + '</div>';
        var el3 = document.querySelector('.iamform');
        el3.outerHTML = '  <form action="https://bbi.us5.list-manage.com/subscribe/post?u=bdb648829adf3af97bbd37b0f&amp;id=6cabd3ccaf&amp;f_id=00d9c2e1f0" method="post" id="mc-embedded-subscribe-form" name="mc-embedded-subscribe-form" class="validate" target="_blank">' + el3.innerHTML + '</form>';
</script>
<%}%>

The code above will:
1)	Remove the initial form tag
2)	Replace the form tag that is removed in the DOM along with the correct action to send the data back to Mailchimp.
3)	It will ensure that when logged in the original DNN Form wrapper is back in place so that administering the site is still possible.


Finally, all you need to do is place the form code into your DNN website, ensuring that you have the new “iamform” div wrapped ready to replace the form tag that is deleted. The example is on page 3.
As for the CSS and JS that is provided by Mailchimp. This can be added in the Module settings of the Text/HTML module. 

 
