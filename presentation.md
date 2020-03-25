title: TDD against the odds
class: animation-fade
layout: true


<!-- This slide will serve as the base layout for all your slides -->

---

class: impact full-width

.impact-wrapper[
# {{title}}
## Three stories
]

???

---

class: center middle

## ThoughtWorks  ❤️   Test Driven Development

---

class: center middle

![tdd](images/tdd.png)

---

class: center middle

## Let's look at an example

---

class: center middle

```kotlin
data class Sum(private val value: Int) {
    operator fun plus(other: Sum): Sum
}
```

--

```kotlin
Sum(3) + Sum(2) // Sum(5)
```

---

class: center middle

```kotlin
@Test
fun `adds two values`() {
    expectThat(Sum(3) + Sum(2))
        .isEqualTo(Sum(5))
}
```

---

class: center middle

```kotlin
@Test
fun `identity property`() {
    expectThat(Sum(3) + Sum(0))
        .isEqualTo(Sum(3))
}
```
---

class: center middle

```kotlin
@Test
fun `associative property`() {
    expectThat(Sum(3) + Sum(2))
        .isEqualTo(Sum(2) + Sum(3))
}
```

---

class: center middle

```kotlin
data class Sum(private val value: Int) {
    operator fun plus(other: Sum): Sum {
        return Sum(value + other.value)
    }
}
```

---

class: center middle

## Reality is a lot messier

---

class: center middle

![3-layers](images/3-layers.png)

---

class: transition

# Frontend

---

class: center middle

![cookery](images/cookery.png)

---

class: center middle

## How do you do TDD in React?

---

class: center middle

.col-8[
![jest](images/jest.png)
]

.col-4[
![enzyme](images/enzyme.jpg)
]

---

class: center middle


```typescript
it('renders a recipe list', () => {
  const wrapper = shallow(<RecipeList {...props} />)
  
  expect(wrapper.find('Recipe').props().toEqual({
    title: 'Pasta Carbonara'
  }))
})
```

---

class: center middle

### kentcdodds.com/blog/why-i-never-use-shallow-rendering

---

class: center middle

![rtl](images/rtl.png)

---

class: center middle

> The more your tests resemble the way your software is used, the more confidence they can give you.

---

class: center middle

```typescript
it('renders a recipe list', async () => {
  const { getByTestId, getByText, getAllByText } = fullRender(<App />, {
    route: '/recipes',
  })
})
```

---

class: center middle

```typescript
it('renders a recipe list', async () => {
  const { getByTestId, getByText, getAllByText } = fullRender(<App />, {
    route: '/recipes',
  })

* // Navigate to list of recipes
* await waitForElement(() => getByTestId('recipe-list'))
})
```

---

class: center middle

```typescript
it('renders a recipe list', async () => {
  const { getByTestId, getByText, getAllByText } = fullRender(<App />, {
    route: '/recipes',
  })

  // Navigate to list of recipes
  await waitForElement(() => getByTestId('recipe-list'))

* // Select one recipe
* await waitForElement(() => getByText('Pasta Carbonara'))
* userEvent.click(getAllByText('DETAILS')[0])
* await waitForElement(() => getByText('egg'))
})
```

---

class: center middle

```typescript
it('renders a recipe list', async () => {
  const { getByTestId, getByText, getAllByText } = fullRender(<App />, {
    route: '/recipes',
  })

  // Navigate to list of recipes
  await waitForElement(() => getByTestId('recipe-list'))

  // Select one recipe
  await waitForElement(() => getByText('Pasta Carbonara'))
  userEvent.click(getAllByText('DETAILS')[0])
  await waitForElement(() => getByText('egg'))

* // Go back
* userEvent.click(getByText('Back'))
* await waitForElement(() => getByTestId('recipe-list'))
})
```

---

class: center middle

```console
PASS  src/App.test.tsx
  App
    ✓ renders without crashing (111ms)
    ✓ renders new recipe form (107ms)
    ✓ renders a recipe list (148ms)
    ✓ renders recipe details (62ms)
    ✓ redirects to recipes list (39ms)

Test Suites: 1 passed, 1 total
Tests:       5 passed, 5 total
Snapshots:   0 total
Time:        5.534s, estimated 10s
Ran all test suites related to changed files.

Watch Usage: Press w to show more.
```

---

class: transition

# Backend

---

class: center middle

![many-apis](images/many-apis.png)

---

class: center middle

![many-apis-mock](images/many-apis-mock.png)

---

class: left middle

## .red[❌] Maintenance

--

## .red[❌] Is it still working?

---

class: center middle

## Alternative
### Recordings

---

class: center middle

![wiremock](images/wiremock.jpg)

---

class: center middle

![recordings](images/recordings.png)

---

class: center middle

## Workflow

---

class: center middle

## Write basic API client that makes a call

---

class: center middle

```kotlin
@Component
class JsonPlaceholder {
    @Autowired
    lateinit var template: RestTemplate

    fun todos(): JsonNode {
        val response = template.exchange(
                "/todos",
                HttpMethod.GET,
                null,
                JsonNode::class.java)
        return response.body
    }
}
```

---

class: center middle

## Write integration test that calls the real endpoint and stores the result

---

class: center middle

```kotlin
@Category(IntegrationTest::class)
@RunWith(SpringRunner::class)
@SpringBootTest
class JsonPlaceholderIntegrationTest : RecordingTest() {
    public override fun recordingServerUrl(): String {
        return "https://jsonplaceholder.typicode.com"
    }

    @Autowired
    lateinit var subject: JsonPlaceholder

    @Test
    fun todos() {
        subject.todos()
    }
}
```

---

class: center middle

## Use recording in unit test and refine call

---

class: center middle

```kotlin
@RunWith(SpringRunner::class)
@SpringBootTest
class JsonPlaceholderTest : RecordedTest() {
    @Autowired
    lateinit var subject: JsonPlaceholder

    @Test
    fun todos() {
        expectThat(subject.todos())
            .isNotEmpty()
    }
}
```

---

class: center middle

### hceris.com/recording-apis-with-wiremock

---

class: transition

# Infrastructure

---

class: center middle

![docker](images/docker.png)

---

class: center middle

![serverspec](images/serverspec.png)

.bottom-right[
serverspec.org
]

---

class: center middle

```ruby
require 'serverspec'
require 'docker'
require 'rspec/wait'

set :backend, :docker
set :docker_image, 'example-openjdk'

RSpec.configure do |config|
  config.wait_timeout = 60 # seconds
end
```

---

class: middle

### OS Version

```ruby
describe file('/etc/alpine-release') do
  its(:content) { is_expected.to match(/3.8.2/) }
end
```

---

class: middle

### Java Version

```ruby
describe command('java -version') do
  its(:stderr) { is_expected.to match(/1.8.0_181/) }
end
```

---

class: middle

### JAR

```ruby
describe file('gs-rest-service.jar') do
  it { is_expected.to be_file }
end
```

---

class: middle

### App is running

```ruby
describe process('java') do
  it { is_expected.to be_running }
  its(:args) { is_expected.to contain('gs-rest-service.jar') }
end
```

---

class: middle

### Bound to right port

```ruby
describe 'listens to correct port' do
  it { wait_for(port(8080)).to be_listening.with('tcp') }
end
```

---

class: middle

### Not running under root

```ruby
describe process('java') do
  its(:user) { is_expected.to eq('runner') }
end
```

---

class: center middle

```Dockerfile
FROM openjdk:8-jre-alpine3.8

WORKDIR /app
EXPOSE 8080
ENV VERSION="0.1.0"

COPY build/libs/gs-rest-service-${VERSION}.jar gs-rest-service.jar

RUN adduser -D runner

USER runner
CMD ["java", "-jar", "gs-rest-service.jar"]
```

---

class: center middle

```console
rspec spec/container_spec.rb
Randomized with seed 61858
.......

Top 7 slowest examples (12.86 seconds, 99.9% of total time):
  Application Container java listens to correct port should be listening with tcp
    7.82 seconds ./spec/container_spec.rb:20
  Application Container java Process "java" should be running
    4.14 seconds ./spec/container_spec.rb:14
  Application Container java Command "java -version" stderr should match /1.8.0_181/
    0.3948 seconds ./spec/container_spec.rb:10
  Application Container java Process "java" args should contain "gs-rest-service.jar"
    0.15328 seconds ./spec/container_spec.rb:15
  (more output ...)

Finished in 12.87 seconds (files took 1.69 seconds to load)
7 examples, 0 failures
```

---

class: center middle

### github.com/sirech/talks/blob/master/2019-01-tw-tdd_containers.pdf

---

class: transition

# Conclusion

---

class: center middle

## TDD can be used in situations you didn't think about

---

class: center middle

## Get creative

---

class: center middle

## Don't get stuck at the definition of a *Unit Test*

---

class: transition

## Mario Fernandez
 
 **Thought**Works

---
