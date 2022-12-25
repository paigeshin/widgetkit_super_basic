# Send data with AppGroup
```
import SwiftUI
import WidgetKit

struct ContentView: View {
    var body: some View {
        VStack {
            Button {
                let defaults = UserDefaults(suiteName: "group.com.paigesoftware.widgettutorial")
                
                defaults?.setValue("hello my widget", forKey: "text")
                WidgetCenter.shared.reloadAllTimelines()
                
            } label: {
                Text("Pass Data")
            }

        }
        .padding()
    }
}

struct ContentView_Previews: PreviewProvider {
    static var previews: some View {
        ContentView()
    }
}

```

### Receiving Data ###

```swift

// Provider is responsible for timeline of widget views
// For a given time frame. (e.g. weather app => 30 seconds interval)
struct Provider: IntentTimelineProvider {
    
    func placeholder(in context: Context) -> SimpleEntry {
        // Dummy Data
        SimpleEntry(date: Date(), text: "", configuration: ConfigurationIntent())
    }

    // ASYNC TASK 
    func getSnapshot(for configuration: ConfigurationIntent, in context: Context, completion: @escaping (SimpleEntry) -> ()) {
        let entry = SimpleEntry(date: Date(), text: "", configuration: ConfigurationIntent())
        completion(entry)
    }

    func getTimeline(for configuration: ConfigurationIntent, in context: Context, completion: @escaping (Timeline<Entry>) -> ()) {
        var entries: [SimpleEntry] = []
        
        // Where to provide..
        let defaults = UserDefaults(suiteName: "group.com.paigesoftware.widgettutorial")
        let text = defaults?.string(forKey: "text") ?? "No Text"
        
        // Generate a timeline consisting of five entries an hour apart, starting from the current date.
        let currentDate = Date()
        for hourOffset in 0 ..< 5 {
            let entryDate = Calendar.current.date(byAdding: .hour, value: hourOffset, to: currentDate)!
            let entry = SimpleEntry(date: entryDate, text: text, configuration: configuration)
            entries.append(entry)
        }

        let timeline = Timeline(entries: entries, policy: .atEnd)
        completion(timeline)
    }
}

struct SimpleEntry: TimelineEntry {
    let date: Date
    let text: String
    let configuration: ConfigurationIntent
}

struct My_WidgetEntryView : View {
    var entry: Provider.Entry

    var body: some View {
        ZStack {
            GeometryReader { geo in
                Image("image")
                    .resizable()
                    .aspectRatio(contentMode: .fill)
                    .frame(width: geo.size.width, height: geo.size.height, alignment: .center)
            }
            Text(entry.text)
                .font(.system(size: 24, weight: .semibold, design: .default))
                .foregroundColor(.white)
        }
    }
}

struct My_Widget: Widget {
    let kind: String = "My_Widget"

    var body: some WidgetConfiguration {
        IntentConfiguration(kind: kind, intent: ConfigurationIntent.self, provider: Provider()) { entry in
            My_WidgetEntryView(entry: entry)
        }
        .configurationDisplayName("My Widget")
        .description("This is an example widget.")
    }
}

struct My_Widget_Previews: PreviewProvider {
    static var previews: some View {
        My_WidgetEntryView(entry: SimpleEntry(date: Date(), text: "Hello Widget!", configuration: ConfigurationIntent()))
            .previewContext(WidgetPreviewContext(family: .systemSmall))
    }
}

```
