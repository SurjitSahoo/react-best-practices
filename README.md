# React Best Practices

## Problem Statement

Bad code is a technical debt 💩. We will have to pay for it eventually. It'll get harder and harder to maintain in the long run. And you'll get to a point where adding more developers to the team has diminishing returns. Insufficient design for any piece of code will remove its ability to change, scale, or to be re-used. But change is inevitable, it'll always happen, It needs to happen. And change relies on existing code's ability to change scale. Good architecture solves these problems.

We will define the best practices from two perspectives.

1. [Architecture](#architecture)
2. [Performance](#performance)

## Architecture

### Separation of Concerns

React is the antithesis of this principle. It has HTML, CSS, JavaScript, all in the same file. What should we do about it? Well don't think of separating things based on categories, separate them based on responsibilities. Components are the building blocks of react. So separate in terms of components. E.g.

```text
button
  ├── button.tsx
  ├── button.css
  ├── button.tests.ts
  ├── button.types.ts
  ├── button.stories.tsx
  └── index.ts <--------- export Button component and ButtonProps type from here
```

- Where is the button component? → In the button directory
- Where are the CSS styles for button component? → In the button directory
- Where are the tests of button component? → In the button directory
- Where are the storybook docs of button component? → In the button directory

You get the idea!

#### How do I structure my project?

While most of the online courses recommend us to use the following folder structure. It's often not very easy to traverse logically. We should also consider the fact that those courses are meant for beginners.

```text
app
├── assets
│   ├── images
│   └── styles
├── components
├── containers or pages
├── hooks
└── utils
```

This structure is confusing because what is the difference between components and containers? You started a component(state less) in the component directory, now the component needs a state, while you can easily add that with hooks, should it be moved to containers now??

All the features are scattered over all the folders. It's not a very good structure

Structure the app keeping "Separation of Concerns" in mind. Consider the following

```text
YouTube
  ├─ commonComponents
  │   └─ inputs
  │       ├── textInput
  │       ├── radio
  │       ├── autoCompleteInput
  │       └── button
  │
  ├─ features
  │   ├── player
  │   │   ├── assets <------------ common assets for player feature
  │   │   ├── defaultPlayer
  │   │   ├── pictureInPicturePlayer
  │   │   │   └── assets <----------- PinP specific assets
  │   │   └── hooks <------------- Player specific hooks
  │   │       └── useCurrentlyPlaying.ts
  │   └── comment
  │       ├── commenting
  │       └── liveChat
  │
  ├─ pages <---------------- pages are constructed using different features
  │   ├── home
  │   ├── profile
  │   └── subscriptions
  │
  ├─ hooks <------------------ global common hooks
  │   ├── useDarkMode.ts
  │   ├── useEffectOnce.ts
  │   ├── useFetch.ts
  │   └── useLocalStorage.ts
  │
  └─ utils
```

This is much more organized! 😀

### Single Responsibility Principle

One "thing" should only concern about doing one thing properly. This "one thing" can be a React Component, a function or a module.

#### Examples

- Components

  If you need a feedback form in a pop-up, the form and the pop-up modal should not be one single component. They should not even be tightly coupled with each other.

  ```jsx
  // Reusable Modal component
  function Modal({ close = ()=>closeModal(), title="Modal Title", children }) {
      return React.createPortal(
          <div>
              <header>
                  <h2>{title}</h2>
                  <button onClick={close}>close</button>
              </header>
              <main>{children}</main>
          </div>
      );
  }

  function FeedbackForm({ onSubmit, onCancel }) {
      ...input states...
      const submit = async () => {
          await onSubmit(formValues);
      }

      const cancel = async () => {
          formCleanup();
          onCancel()
      }

      return (
          <form>
              ...form inputs
          </form>
      );
  }

  function FeedbackPopUpFeature() {
      const [isModalOpen, setIsModalOpen] = useState(false);
      const closeModal = () => setIsModalOpen(false);
      const openModal = () => setIsModalOpen(true);

      const handleSubmit = formValues => {
          await api.post(formValues);
          closeModal();
      }

      return (
          <>
              <button onClick={openModal}>Provide feedback</button>
              {isModalOpen && (
                  <Modal title="Provide your valuable feedback" close={closeModal} >
                      <FeedbackForm onCancel={closeModal} onSubmit={handleSubmit} />
                  </Modal>
              )}
          </>
      );
  }
  ```

##### What's good about this design?

- The **modal component** doesn't care about anything other than being a modal. It only needs to do one thing properly, that is being a modal. How to properly render the modal, how to close it, the styles for the modal, and so on. The modal will be opened by the parent, but it'll be closed by the modal itself, so it takes a close function as arg.
- **FeedbackForm component** only deals with handling the form, cleaning it up and calling the submit function with necessary form input values. This component does not need to know where it will be rendered, whether to open/close the wrapping modal and so on. In order to use this modal component in other use cases, the modal itself should not need to be changed at all.
- And lastly the **feedbackPopUpFeature** combines the modal and the form component to produce the desired feature. You can even take it one step further and create all the inputs as separate components.
- Just like the modal and the feedbackForm components all the react components should be individually reusable. They should not be tightly coupled at all. So that we can always mix and match multiple components together to produce new features.
