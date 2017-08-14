---
layout: post
title: "Server Sent Events (SSE) with Spring Framework (Pt. II)"
date: 2017-08-14 00:00:00 +0200
comments: true
category: blog
tags: [java, spring, sse, html5]
---

In the [first part]({% post_url 2017-06-26-server-sent-events-with-spring-framework-part-I %}) of this series of articles about SSE and Spring I talked about the concept of SSE, why SSE would be a good choice and the typical examples where SSE fits good. Additionally, I described the main stuff involved in Spring to implement SSE in your project.
<!-- more -->

In this article I want to explain my use case and how we can add support for SSE in a Spring Boot project seamlessly, so that it would be very straightforward to add more SSE channels in your project whenever needed. 

## Use Case
One of the requirements of the project was to build an items feed in real time. As soon as the server finds a new item it has to push it to the client. The client simply opens a channel and keeps listening the stream of updates. Since the client doesn't need to send any data, we don't need bi-directional communication, so SSE becomes a very good solution.

This application is basically a REST API built using **Spring Boot 1.5.3**. Below are described the different components involved.
[The source code can be found in GitHub](https://github.com/rfvallina/items-feed)


### SseEngine

SSE Engine will be responsible for calling the proper handler in order to get the necessary data to be sent. There are two important methods in this class. One is `runInLoop` which keeps calling the handler until the timeout reaches. The other method is `run` which calls the handler once. Note that both methods are annotated with `@Async` which means they will run asynchronosuly.

```java

@Component
public class SseEngine {

	private static Logger logger = LoggerFactory.getLogger(SseEngine.class);

	private static long TIMEOUT = 30000L;

	private final Map<String, SseEmitter> emitters = new ConcurrentHashMap<>();

	/**
	 * Setup SSE emitter and keep publishing data until timeout completes
	 * 
	 * @param sse
	 * @param data
	 */
	@Async
	public void runInLoop(Sse sseService, SseData data) {
		config(emitters.get(data.getEventId()), data.getEventId());

		if(emitters.get(data.getEventId()) != null){
			while ((Calendar.getInstance().getTimeInMillis() - data.getStarted().getTime()) < TIMEOUT) {
				sseService.handle(data);
				try {
					Thread.sleep(500);
				} catch (InterruptedException e) {
					logger.error(e.getMessage());
				}
			}
		}
		
	}

	/**
	 * Setup SSE emitter and publish data
	 * 
	 * @param sse
	 * @param eventData
	 */
	@Async
	public void run(Sse sseService, SseData eventData) {
		if(emitters.get(eventData.getEventId()) != null){
			config(emitters.get(eventData.getEventId()), eventData.getEventId());
			sseService.handle(eventData);
		}
	}

	private void config(SseEmitter emitter, String eventId) {
		if (emitter != null) {
			emitter.onCompletion(() -> {
				logger.debug("Emitter " + emitter.toString() + " COMPLETED!");
				emitters.remove(eventId);
			});
			emitter.onTimeout(() -> {
				logger.debug("Emitter " + emitter.toString() + " TIMEOUT!");
				emitters.remove(eventId);
			});
		}
	}

	public long getTimeout() {
		return TIMEOUT;
	}

	public Map<String, SseEmitter> getEmitters() {
		return emitters;
	}
}
```

### Sse

This is an interface whose aim is to be implemented by the service responsible of getting the data to be sent through the channel. It has 3 methods.

- `public void push(String eventId, Object data)`

This is basically the initiator of the SSE chain. This will be typically invoked by the controller which will give the order to start the mechanism of getting data, publish, listen and then send through the emitter.

- `public void handle(SseData eventData)`

This method will have the business logic to get the required data to be sent through the emitter. In our example this method is the responsible for finding the items.

- `@EventListener
public void onPublish(AbstractApplicationEvent event)`

This is the event listener. By default it will try to listen any type of event. Typically, in our implementation we will restrict the listening to our specific event class having the items. In the future, if we would need another SSE channel (i.e. for notifications) we would create a new event class like **NotificationsEvent** to listen only those events.

```java

public interface Sse {
	
	/**
	 * Push data to SSE channel associated to an event ID
	 * 
	 * @param eventId
	 * @param data
	 */
	public void push(String eventId, Object data);

	/**
	 * This method does the necessary actions to publish the event with the proper data
	 * 
	 * @param eventData
	 */
	public void handle(SseData eventData);

	/**
	 * This method is listening for new events being published. To restrict the events listened to only a specific group, the event classes in
	 * charge of handle the events published must be specified within the annotation @EventListener
	 * 
	 * @param event
	 */
	@EventListener
	public void onPublish(AbstractApplicationEvent event);
}


```

### SseData

This is an abstract class to be extended by our specific class containing the necessary to monitor the cycle and the different executions. This class has an `eventId` which is the unique identifier for the channel opened by the client, so that the process doesn't send data to the wrong client.

```java

public abstract class SseData {

	protected String eventId;
	protected Date started;
	
	public String getEventId() {
		return eventId;
	}

	public void setEventId(String eventId) {
		this.eventId = eventId;
	}

	public Date getStarted() {
		return started;
	}

	public void setStarted(Date started) {
		this.started = started;
	}
	
}

```

### ItemsFeedSseService

This is our implementation of the Sse interface that we saw above. As you see, `handle` method loads the items (from a json file in our example) and publishes a new event using `ApplicationEventPublisher` class from Spring framework which publishes the event to be listened by the our event listener.
Our event listener is annotated with `@EventListener(classes = ResultsEvent.class)` which means it will listen only those events of type `ResultsEvent`. It gets the items, finds the proper emitter associated with our eventId and sends the data through the emitter.

```java

@Service
public class ItemsFeedSseService implements Sse {

	private static Logger logger = LoggerFactory.getLogger(ItemsFeedSseService.class);

	@Autowired
	SseEngine sseEngine;

	@Autowired
	private ApplicationEventPublisher applicationEventPublisher;

	@Override
	public void push(String eventId, Object data) {
		sseEngine.runInLoop(this, new ResultsSseData(eventId));
	}

	@Override
	public void handle(SseData eventData) {
		ResultsSseData serviceData = (ResultsSseData) eventData;
		List<Item> items = loadItems();
		int nextIndex = serviceData.getLastIndex() + 1;
		if(items != null && nextIndex < items.size()){
			Item item = items.get(nextIndex);
			serviceData.getProcessedItems().add(item);
			serviceData.setLastIndex(nextIndex);
			try {
				// sleep 1 second for the first element, 2 seconds for the second,... 
				Thread.sleep((nextIndex + 1) * 1000);
			} catch (InterruptedException e) {
				logger.error(e.getMessage());
			}
			
			applicationEventPublisher.publishEvent(new ResultsEvent(this, serviceData.getEventId(), Arrays.asList(item)));
		}
	}

	@Override
	@EventListener(classes = ResultsEvent.class)
	public void onPublish(AbstractApplicationEvent event) {
		ResultsEvent resultsEvent = (ResultsEvent) event;
		SseEmitter emitter = sseEngine.getEmitters().get(event.getEventId());
		try {

			if (!resultsEvent.getItems().isEmpty()) {
				logger.debug("Sending message through emmitter " + emitter.toString());
				emitter.send(resultsEvent.getItems());
			}

		} catch (IOException e) {
			logger.error("Error in emitter " + emitter + " while sending message");
			sseEngine.getEmitters().remove(event.getEventId());
		}

	}
	
	private List<Item> loadItems(){
		List<Item> items = null;
		ObjectMapper om = new ObjectMapper();
		try {
			items = om.readValue(new File("src/main/resources/json/items.json"), new com.fasterxml.jackson.core.type.TypeReference<List<Item>>(){});
		} catch (Exception e) {
			logger.error("Error parsing JSON file - " + e.getMessage());
		}
		return items;
	}

}

```

### FeedController

This is annotated with the Spring annotation `@Controller` which tells Spring that this will act as a controller for the incoming http requests. We inject the Sse engine and the proper service to fetch the data. Basically, as a new request comes in, it creates a new event Id and invokes `push` method of the Sse service to trigger the SSE mechanism as we saw before.
Note that the method returns an `SseEmitter` which forces the response to return **text/event-stream** Content-Type header.

```java

@Controller
@CrossOrigin
public class FeedController {
	
	@Autowired
	SseEngine sseEngine;
	
	@Autowired
	ItemsFeedSseService sseService;

	@GetMapping("/feed")
	public SseEmitter getResults() {
		String eventId = UUID.randomUUID().toString();
		SseEmitter emitter = new SseEmitter(sseEngine.getTimeout());
		sseEngine.getEmitters().put(eventId, emitter);

		// push notifications
		sseService.push(eventId, null);

		return emitter;
	}
}

```


In this example we saw how to implement an items feed using SSE and Spring. [The source code can be found in GitHub](https://github.com/rfvallina/items-feed)

## Testing

You can test this application easily by using the below html. Once you start the application locally, then load the below code in a browser and you will see items coming in. As you see there's some javascript code which starts a connection to our API endpoint using `EventSource` object. The client keeps listening and the server keeps updating the channel with new items.
You can find this html file under the html folder in the project as well.

```html

<!DOCTYPE html>
<html>
<script src="https://code.jquery.com/jquery-3.2.1.min.js" integrity="sha256-hwg4gsxgFZhOsEEamdOYGBf13FyQuiTwlAQgxVSNgt4="
	crossorigin="anonymous"></script>
<script type="text/javascript">
	$(document).ready(function() {
		var source;
		var url = 'http://localhost:8080/feed';

		if (!!window.EventSource) {
			source = new EventSource(url);
			source.addEventListener('message', function(e) {
				var data = JSON.parse(e.data);
				if (data && data.length > 0) {
					$("#data tbody").append("<tr><td>" + data[0].itemId + "</td><td>" + data[0].title + "</td><td>" + data[0].currentPrice + "</td></tr>");
				} else
					console.log("No items found");
			}, true);

			source.addEventListener('open', function(e) {
				console.log("Connection opened");
			}, true);

			source.addEventListener('error', function(e) {
				console.log("Connection error");
			}, true);

			source.addEventListener('close', function(e) {
				console.log("Connection closed");
			}, true);

			
		}
		
	});
</script>
<body>
	<h3>Items Feed</h3>
	
	<div id="data">
		<table border="1">
			<thead>
				<tr>
					<td>Id</td>
					<td>Title</td>
					<td>Price</td>
				</tr>
			</thead>
			<tbody></tbody>
		</table>
	</div>
</body>
</html>

```

