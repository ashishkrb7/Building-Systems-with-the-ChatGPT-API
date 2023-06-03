# Build an End-to-End System

## Setup
#### Load the API key and relevant Python libaries.
Some code that loads the OpenAI API key for you.


```python
import os
import openai
import sys
import utils

import panel as pn  # GUI
pn.extension()

from dotenv import load_dotenv, find_dotenv
_ = load_dotenv(find_dotenv()) # read local .env file

openai.api_type = os.getenv("api_type")
openai.api_base = os.getenv("api_base")
openai.api_version = os.getenv("api_version")
openai.api_key = os.getenv("OPENAI_API_KEY")
```






<style>.bk-root, .bk-root .bk:before, .bk-root .bk:after {
  font-family: var(--jp-ui-font-size1);
  font-size: var(--jp-ui-font-size1);
  color: var(--jp-ui-font-color1);
}
</style>



```python
def get_completion_from_messages(messages, model="chatgpt-gpt35-turbo", temperature=0, max_tokens=500):
    response = openai.ChatCompletion.create(
        engine=model,
        messages=messages,
        temperature=temperature, 
        max_tokens=max_tokens, 
    )
    return response.choices[0].message["content"]
```

## System of chained prompts for processing the user query


```python
def process_user_message(user_input, all_messages, debug=True):
    delimiter = "```"
    
#     # Step 1: Check input to see if it flags the Moderation API or is a prompt injection
#     response = openai.Moderation.create(input=user_input)
#     moderation_output = response["results"][0]

#     if moderation_output["flagged"]:
#         print("Step 1: Input flagged by Moderation API.")
#         return "Sorry, we cannot process this request."

    if debug: print("Step 1: Input passed moderation check.")
    
    category_and_product_response = utils.find_category_and_product_only(user_input, utils.get_products_and_category())
    #print(print(category_and_product_response)
    # Step 2: Extract the list of products
    category_and_product_list = utils.read_string_to_list(category_and_product_response)
    #print(category_and_product_list)

    if debug: print("Step 2: Extracted list of products.")

    # Step 3: If products are found, look them up
    product_information = utils.generate_output_string(category_and_product_list)
    if debug: print("Step 3: Looked up product information.")

    # Step 4: Answer the user question
    system_message = f"""
    You are a customer service assistant for a large electronic store. \
    Respond in a friendly and helpful tone, with concise answers. \
    Make sure to ask the user relevant follow-up questions.
    """
    messages = [
        {'role': 'system', 'content': system_message},
        {'role': 'user', 'content': f"{delimiter}{user_input}{delimiter}"},
        {'role': 'assistant', 'content': f"Relevant product information:\n{product_information}"}
    ]

    final_response = get_completion_from_messages(all_messages + messages)
    if debug:print("Step 4: Generated response to user question.")
    all_messages = all_messages + messages[1:]

#     # Step 5: Put the answer through the Moderation API, I am using AzureOpenAI and and I don't have access to moderation api
#     response = openai.Moderation.create(input=final_response)
#     moderation_output = response["results"][0]

#     if moderation_output["flagged"]:
#         if debug: print("Step 5: Response flagged by Moderation API.")
#         return "Sorry, we cannot provide this information."

    if debug: print("Step 5: Response passed moderation check.")

    # Step 6: Ask the model if the response answers the initial user query well
    user_message = f"""
    Customer message: {delimiter}{user_input}{delimiter}
    Agent response: {delimiter}{final_response}{delimiter}

    Does the response sufficiently answer the question?
    """
    messages = [
        {'role': 'system', 'content': system_message},
        {'role': 'user', 'content': user_message}
    ]
    evaluation_response = get_completion_from_messages(messages)
    if debug: print("Step 6: Model evaluated the response.")

    # Step 7: If yes, use this answer; if not, say that you will connect the user to a human
    if "Y" in evaluation_response:  # Using "in" instead of "==" to be safer for model output variation (e.g., "Y." or "Yes")
        if debug: print("Step 7: Model approved the response.")
        return final_response, all_messages
    else:
        if debug: print("Step 7: Model disapproved the response.")
        neg_str = "I'm unable to provide the information you're looking for. I'll connect you with a human representative for further assistance."
        return neg_str, all_messages

user_input = "tell me about the smartx pro phone and the fotosnap camera, the dslr one. Also what tell me about your tvs"
response,_ = process_user_message(user_input,[])
print(response)
```

    Step 1: Input passed moderation check.
    Step 2: Extracted list of products.
    Step 3: Looked up product information.
    Step 4: Generated response to user question.
    Step 5: Response passed moderation check.
    Step 6: Model evaluated the response.
    Step 7: Model approved the response.
    The SmartX ProPhone is a powerful smartphone with a 6.1-inch display, 128GB storage, 12MP dual camera, and 5G capabilities. The FotoSnap DSLR Camera is a versatile camera with a 24.2MP sensor, 1080p video, 3-inch LCD, and interchangeable lenses. As for our TVs, we have a range of options including the CineView 4K TV with a 55-inch display, 4K resolution, HDR, and smart TV capabilities, the CineView 8K TV with a 65-inch display, 8K resolution, HDR, and smart TV capabilities, and the CineView OLED TV with a 55-inch display, 4K resolution, HDR, and smart TV capabilities. Do you have any specific questions about these products or would you like me to recommend a product based on your needs?
    

### Function that collects user and assistant messages over time


```python
def collect_messages(debug=False):
    user_input = inp.value_input
    if debug: print(f"User Input = {user_input}")
    if user_input == "":
        return
    inp.value = ''
    global context
    #response, context = process_user_message(user_input, context, utils.get_products_and_category(),debug=True)
    response, context = process_user_message(user_input, context, debug=False)
    context.append({'role':'assistant', 'content':f"{response}"})
    panels.append(
        pn.Row('User:', pn.pane.Markdown(user_input, width=600)))
    panels.append(
        pn.Row('Assistant:', pn.pane.Markdown(response, width=600, style={'background-color': '#F6F6F6'})))
 
    return pn.Column(*panels)
```

### Chat with the chatbot!
Note that the system message includes detailed instructions about what the OrderBot should do.


```python
panels = [] # collect display 

context = [ {'role':'system', 'content':"You are Service Assistant"} ]  

inp = pn.widgets.TextInput( placeholder='Enter text here…')
button_conversation = pn.widgets.Button(name="Service Assistant")

interactive_conversation = pn.bind(collect_messages, button_conversation)

dashboard = pn.Column(
    inp,
    pn.Row(button_conversation),
    pn.panel(interactive_conversation, loading_indicator=True, height=300),
)

dashboard
```






<div id='1002'>
  <div class="bk-root" id="1da83859-ddc7-4b29-b3a9-9b4b6f7fc6b5" data-root-id="1002"></div>
</div>
<script type="application/javascript">(function(root) {
  function embed_document(root) {
    var docs_json = {"6595baa5-0dc5-4cca-948c-a0c69d5da08b":{"defs":[{"extends":null,"module":null,"name":"ReactiveHTML1","overrides":[],"properties":[]},{"extends":null,"module":null,"name":"FlexBox1","overrides":[],"properties":[{"default":"flex-start","kind":null,"name":"align_content"},{"default":"flex-start","kind":null,"name":"align_items"},{"default":"row","kind":null,"name":"flex_direction"},{"default":"wrap","kind":null,"name":"flex_wrap"},{"default":"flex-start","kind":null,"name":"justify_content"}]},{"extends":null,"module":null,"name":"GridStack1","overrides":[],"properties":[{"default":"warn","kind":null,"name":"mode"},{"default":null,"kind":null,"name":"ncols"},{"default":null,"kind":null,"name":"nrows"},{"default":true,"kind":null,"name":"allow_resize"},{"default":true,"kind":null,"name":"allow_drag"},{"default":[],"kind":null,"name":"state"}]},{"extends":null,"module":null,"name":"click1","overrides":[],"properties":[{"default":"","kind":null,"name":"terminal_output"},{"default":"","kind":null,"name":"debug_name"},{"default":0,"kind":null,"name":"clears"}]},{"extends":null,"module":null,"name":"NotificationAreaBase1","overrides":[],"properties":[{"default":"bottom-right","kind":null,"name":"position"},{"default":0,"kind":null,"name":"_clear"}]},{"extends":null,"module":null,"name":"NotificationArea1","overrides":[],"properties":[{"default":[],"kind":null,"name":"notifications"},{"default":"bottom-right","kind":null,"name":"position"},{"default":0,"kind":null,"name":"_clear"},{"default":[{"background":"#ffc107","icon":{"className":"fas fa-exclamation-triangle","color":"white","tagName":"i"},"type":"warning"},{"background":"#007bff","icon":{"className":"fas fa-info-circle","color":"white","tagName":"i"},"type":"info"}],"kind":null,"name":"types"}]},{"extends":null,"module":null,"name":"Notification","overrides":[],"properties":[{"default":null,"kind":null,"name":"background"},{"default":3000,"kind":null,"name":"duration"},{"default":null,"kind":null,"name":"icon"},{"default":"","kind":null,"name":"message"},{"default":null,"kind":null,"name":"notification_type"},{"default":false,"kind":null,"name":"_destroyed"}]},{"extends":null,"module":null,"name":"TemplateActions1","overrides":[],"properties":[{"default":0,"kind":null,"name":"open_modal"},{"default":0,"kind":null,"name":"close_modal"}]},{"extends":null,"module":null,"name":"MaterialTemplateActions1","overrides":[],"properties":[{"default":0,"kind":null,"name":"open_modal"},{"default":0,"kind":null,"name":"close_modal"}]}],"roots":{"references":[{"attributes":{"margin":[5,10,5,10],"max_length":5000,"placeholder":"Enter text here\u2026"},"id":"1003","type":"TextInput"},{"attributes":{"icon":null,"js_event_callbacks":{"button_click":[{"id":"1008"}]},"label":"Service Assistant","margin":[5,10,5,10],"subscribed_events":["button_click"]},"id":"1005","type":"Button"},{"attributes":{"children":[{"id":"1003"},{"id":"1004"},{"id":"1006"}],"margin":[0,0,0,0],"name":"Column00110"},"id":"1002","type":"Column"},{"attributes":{"reload":false},"id":"1010","type":"panel.models.location.Location"},{"attributes":{"children":[{"id":"1007"}],"height":300,"margin":[0,0,0,0],"min_height":300,"name":"Row00108"},"id":"1006","type":"Row"},{"attributes":{"children":[{"id":"1005"}],"margin":[0,0,0,0],"name":"Row00103"},"id":"1004","type":"Row"},{"attributes":{"margin":[5,5,5,5],"name":"Str00106","text":"&lt;pre&gt; &lt;/pre&gt;"},"id":"1007","type":"panel.models.markup.HTML"},{"attributes":{"client_comm_id":"0d5d05c8186c4373844ad2fa3c536379","comm_id":"1ecd1eb844b841f481fce8f69175d286","plot_id":"1002"},"id":"1009","type":"panel.models.comm_manager.CommManager"},{"attributes":{"args":{"bidirectional":false,"properties":{"event:button_click":"loading"},"source":{"id":"1005"},"target":{"id":"1006"}},"code":"\n    if ('event:button_click'.startsWith('event:')) {\n      var value = true\n    } else {\n      var value = source['event:button_click'];\n      value = value;\n    }\n    if (typeof value !== 'boolean' || source.labels !== ['Loading']) {\n      value = true\n    }\n    var css_classes = target.css_classes.slice()\n    var loading_css = ['pn-loading', 'arc']\n    if (value) {\n      for (var css of loading_css) {\n        if (!(css in css_classes)) {\n          css_classes.push(css)\n        }\n      }\n    } else {\n     for (var css of loading_css) {\n        var index = css_classes.indexOf(css)\n        if (index > -1) {\n          css_classes.splice(index, 1)\n        }\n      }\n    }\n    target['css_classes'] = css_classes\n    ","tags":[[1656163545584,[null,"event:button_click"],[null,"loading"]]]},"id":"1008","type":"CustomJS"}],"root_ids":["1002","1009","1010"]},"title":"Bokeh Application","version":"2.4.3"}};
    var render_items = [{"docid":"6595baa5-0dc5-4cca-948c-a0c69d5da08b","root_ids":["1002"],"roots":{"1002":"1da83859-ddc7-4b29-b3a9-9b4b6f7fc6b5"}}];
    root.Bokeh.embed.embed_items_notebook(docs_json, render_items);
    for (const render_item of render_items) {
      for (const root_id of render_item.root_ids) {
	const id_el = document.getElementById(root_id)
	if (id_el.children.length && (id_el.children[0].className === 'bk-root')) {
	  const root_el = id_el.children[0]
	  root_el.id = root_el.id + '-rendered'
	}
      }
    }
  }
  if (root.Bokeh !== undefined && root.Bokeh.Panel !== undefined) {
    embed_document(root);
  } else {
    var attempts = 0;
    var timer = setInterval(function(root) {
      if (root.Bokeh !== undefined && root.Bokeh.Panel !== undefined) {
        clearInterval(timer);
        embed_document(root);
      } else if (document.readyState == "complete") {
        attempts++;
        if (attempts > 200) {
          clearInterval(timer);
          console.log("Bokeh: ERROR: Unable to run BokehJS code because BokehJS library is missing");
        }
      }
    }, 25, root)
  }
})(window);</script>




```python

```
