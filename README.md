# loopback4-helmet

[![LoopBack](<https://github.com/strongloop/loopback-next/raw/master/docs/site/imgs/branding/Powered-by-LoopBack-Badge-(blue)-@2x.png>)](http://loopback.io/)

A simple loopback-next extension for [helmetjs](https://helmetjs.github.io/) integration in loopback applications.

## Install

```sh
npm install loopback4-helmet
```

## Usage

In order to use this component into your LoopBack application, please follow below steps.

- Add component to application.

```ts
this.component(Loopback4HelmetComponent);
```

- By default, helmet will be initialized with only the default middlewares enabled as per [here](https://github.com/helmetjs/helmet#how-it-works). However, you can override any of the middleware settings using the Config Binding like below.

```ts
this.bind(HelmetSecurityBindings.CONFIG).to({
  referrerPolicy: {
    policy: 'same-origin',
  },
  contentSecurityPolicy: {
    directives: {
      frameSrc: ["'self'"],
    },
  },
});
```

- The component exposes a sequence action which can be added to your server sequence class. Adding this will trigger helmet middleware for all the requests passing through.

```ts
export class MySequence implements SequenceHandler {
  constructor(
    @inject(SequenceActions.FIND_ROUTE) protected findRoute: FindRoute,
    @inject(SequenceActions.PARSE_PARAMS) protected parseParams: ParseParams,
    @inject(SequenceActions.INVOKE_METHOD) protected invoke: InvokeMethod,
    @inject(SequenceActions.SEND) public send: Send,
    @inject(SequenceActions.REJECT) public reject: Reject,
    @inject(HelmetSecurityBindings.HELMET_SECURITY_ACTION)
    protected helmetAction: HelmetAction,
  ) {}

  async handle(context: RequestContext) {
    const requestTime = Date.now();
    try {
      const {request, response} = context;
      const route = this.findRoute(request);
      const args = await this.parseParams(request, route);

      // Helmet Action here
      await this.helmetAction(request, response);

      const result = await this.invoke(route, args);
      this.send(response, result);
    } catch (err) {
      ...
    } finally {
      ...
    }
  }
}
```