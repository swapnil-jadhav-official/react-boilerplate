# react-boilerplate- Webapp
This is to reference for initializing new application in react eject mode. 


## A. Folder Structure (based on create-react-app eject mode)

#### top-level directory layout

    .
    ├── .config                 # create-react-app ejected config
    ├── .env                    # to store environment files
    │   ├── .env.dev            # env for dev server
    │   ├── .env.test           # env for test server
    │   └── .env.uat            # env for uat server
    ├── .scripts                # create-react-app ejected scripts
    ├── .storybook              # storybook config/setup files
    ├── public
    │   ├── assets              # static assets like fonts, js and css
    │   ├── index.html          # root html file to bootstrap the react app 
    ├── src
    │   ├── apis                # to store all the api endpoints as a function 
    │   ├── assets              # to store all the static assets like image/pdf etc
    │   ├── components          # to store all low level components (stateless)
    │   ├── constants           # to store all the constants
    │   │   ├── enums.js        # to store repeated hardcoded array/enums
    │   │   ├── index.js        # to store all the primitive values
    │   │   ├── messages.js     # to store all the message strings
    │   ├── contexts            # reusable global context files
    │   ├── hooks               # reusable global custom hooks
    │   ├── layout              # store components related to layout like DashboardLayout
    │   ├── mockdata            # to store the mock api endpoints until the real ones are ready   
    │   ├── module              # to store the statefull components (containers)
    │   ├── pages               # to store all the page level components, ideally one page for each route
    │   ├── routes              # to store the routes of the application
    │   ├── services            # to store reusable services like storage or notification
    │   ├── stories             # to store all the stories for a components
    │   ├── utils               # to store the utility functions
    │   ├── index.css           # main entry for css, ideally use to  include tailwind or any other third party css modules
    │   ├── index.js            # main entry point of the application
    ├── .gitignore              # files or directory path to ignore on git
    ├── .prettierignore         # files or directory path to ignore on prettier formating
    ├── .package.json           # store all the  information related to the project like dependencies and configurations
    ├── postcss.config.css      # css config required for tailwindcss
    ├── tailwind.config.js      # tailwind customisation file to override or extend the configuration 
    └── README.md


## B. Create New Page

### B1. Sample Page

#### 1. Create following Files 

  - `src/pages/root/sample.page/index.js`
  - `src/pages/root/sample.page/provider.js`

##### `src/pages/root/sample.page/index.js`
```javascript
import Button from "../../../components/button";
import { HeaderRight } from "../../../layouts/dashboard/header";
import withHOC from "../../../utils/with-hoc";
import { SamplePageProvider, useSamplePageContext } from "./provider";

function SamplePage() {
  const { handleIncrementClick, handleDecrementClick, count } = useSamplePageContext();

  return (
    <div>
      <HeaderRight>Sample Page</HeaderRight>
      <div>
        <Button onClick={handleDecrementClick}>Decrement</Button>
        <span>{count}</span>
        <Button onClick={handleIncrementClick}>Increment</Button>
      </div>
    </div>
  );
}

export default withHOC(SamplePageProvider, SamplePage);
```

##### `src/pages/root/sample.page/provider.js`
```javascript
import { useMemo } from "react";
import { useCallback } from "react";
import { useState } from "react";
import generateContext from "../../../utils/generate-context";

const useSamplePage = () => {
  const [count, setCount] = useState(0);

  const handleDecrementClick = useCallback(() => {
    setCount((c) => c - 1);
  }, []);

  const handleIncrementClick = useCallback(() => {
    setCount((c) => c + 1);
  }, []);

  return useMemo(() => {
    return { count, handleDecrementClick, handleIncrementClick };
  }, [count, handleDecrementClick, handleIncrementClick]);
};
export const [SamplePageProvider, useSamplePageContext] = generateContext(useSamplePage);
```

## 2. Add sample route in `routes/public.route.js` file
```javascript
...
    { path: "/sample", element: SamplePage },
...
```

### B2. Sample Page with Sample Form

#### 1. Create following Files 

  - `src/pages/root/sample.page/index.js`
  - `src/pages/root/sample.page/provider.js`
  - `src/pages/root/sample.page/components/sample-form/index.js`
  - `src/pages/root/sample.page/components/sample-form/form-helpers/core.js`
  - `src/pages/root/sample.page/components/sample-form/form-helpers/generate-payload.js`
  - `src/pages/root/sample.page/components/sample-form/form-helpers/get-prefilled-values.js`
  - `src/pages/root/sample.page/components/sample-form/form-helpers/get-form-validation-schema.js`

##### `src/pages/root/sample.page/index.js`
```javascript
import { HeaderRight } from "../../../layouts/dashboard/header";
import withHOC from "../../../utils/with-hoc";
import SampleForm from "./components/sample-form";
import { SamplePageProvider } from "./provider";

function SamplePage() {
  return (
    <div>
      <HeaderRight>Sample Page</HeaderRight>
      <div className="bg-secondary-0 ml-4 mt-4"> 
        <SampleForm />
      </div>
    </div>
  );
}

export default withHOC(SamplePageProvider, SamplePage);
```

##### `src/pages/root/sample.page/provider.js`

```javascript
import { useAsync } from "@react-org/hooks";
import { useCallback, useMemo } from "react";
import { getSampleData, saveSampleData } from "../../../mockdata/samples.mockdata";
import generateContext from "../../../utils/generate-context";
import generatePayload from "./components/sample-form/form-helpers/generate-payload";
import getFormValidationSchema from "./components/sample-form/form-helpers/get-form-validation-schema";
import getPrefilledValues from "./components/sample-form/form-helpers/get-prefilled-values";

const useSamplePage = () => {
  const getCompanyDataQuery = useAsync(getSampleData);
  const saveCompanyDataQuery = useAsync(saveSampleData, { immediate: false });

  const initialValues = useMemo(() => {
    return getPrefilledValues(getCompanyDataQuery?.data?.data);
  }, [getCompanyDataQuery?.data?.data]);

  const validationSchema = useMemo(() => {
    return getFormValidationSchema();
  }, []);

  const handleFormSubmit = useCallback(
    (formValues, { setSubmitting }) => {
      const payload = generatePayload(formValues);
      const saveCompanyDataQueryExecute = saveCompanyDataQuery.execute;
      saveCompanyDataQueryExecute(payload, {
        onError: (err) => {
          console.log("onError", err);
        },
        onSuccess: (res) => {
          console.log("onSuccess", res);
        },
        onComplete: () => {
          console.log("onComplete");
          setSubmitting(false);
        },
      });
    },
    [saveCompanyDataQuery.execute]
  );

  return useMemo(() => {
    return {
      initialValues,
      validationSchema,
      handleFormSubmit,
      getCompanyDataQuery,
    };
  }, [getCompanyDataQuery, handleFormSubmit, initialValues, validationSchema]);
};

export const [SamplePageProvider, useSamplePageContext] = generateContext(useSamplePage);

```

##### `src/pages/root/sample.page/components/sample-form/index.js`

```javascript
import { Form, Formik } from "formik";
import Button from "../../../../../components/button";
import { FormikDatepicker } from "../../../../../components/datepicker";
import FormControl from "../../../../../components/form-control";
import { FormikInput } from "../../../../../components/input";
import InputLabel from "../../../../../components/input-label";
import { FormikTextarea } from "../../../../../components/textarea";
import { useSamplePageContext } from "../../provider";
import { FN } from "./form-helpers/core";

export default function SampleForm() {
  const { initialValues, validationSchema, handleFormSubmit } = useSamplePageContext();
  return (
    <Formik initialValues={initialValues} validationSchema={validationSchema} onSubmit={handleFormSubmit}>
      {(formik) => {
        return (
          <Form>
            <FormControl>
              <InputLabel htmlFor={FN.COMPANY_NAME} required>
                Company Name
              </InputLabel>
              <FormikInput id={FN.COMPANY_NAME} name={FN.COMPANY_NAME} />
            </FormControl>
            <FormControl>
              <InputLabel htmlFor={FN.COMPANY_ADDRESS} required>
                Company Address
              </InputLabel>
              <FormikTextarea id={FN.COMPANY_ADDRESS} name={FN.COMPANY_ADDRESS} />
            </FormControl>
            <FormControl>
              <InputLabel htmlFor={FN.SOME_DATE_FIELD} required>
                Some date field
              </InputLabel>
              <FormikDatepicker id={FN.SOME_DATE_FIELD} name={FN.SOME_DATE_FIELD} />
            </FormControl>
            <div>
              <Button color="primary" type="submit" disabled={formik.isSubmitting}>
                Submit
              </Button>
            </div>
          </Form>
        );
      }}
    </Formik>
  );
}

```

##### `src/pages/root/sample.page/components/sample-form/form-helpers/core.js`

```javascript
export const FN = {
  COMPANY_NAME: "company-name",
  COMPANY_ADDRESS: "company-address",
  SOME_DATE_FIELD: "some-date-field",
};
```

##### `src/pages/root/sample.page/components/sample-form/form-helpers/generate-payload.js`

```javascript
import { getFormattedDateforAPI } from "../../../../../../utils/date-and-time";
import { FN } from "./core";

export default function generatePayload(data = {}) {
  return {
    company_name: data[FN.COMPANY_NAME],
    company_address: data[FN.COMPANY_ADDRESS],
    some_date_field: getFormattedDateforAPI(data[FN.SOME_DATE_FIELD]),
  };
}

```

##### `src/pages/root/sample.page/components/sample-form/form-helpers/get-prefilled-values.js`

```javascript
import { getFormattedDateforUI } from "../../../../../../utils/date-and-time";
import { FN } from "./core";

export default function getPrefilledValues(data = {}){
  return {
    [FN.COMPANY_NAME]: data.company_name || "",
    [FN.COMPANY_ADDRESS]: data.company_address || "",
    [FN.SOME_DATE_FIELD]: getFormattedDateforUI(data.some_date_field),
  };
};
```

##### `src/pages/root/sample.page/components/sample-form/form-helpers/get-form-validation-schema.js`

```javascript
import * as yup from "yup";
import * as validation from "../../../../../../utils/validation";
import { FN } from "./core";

export default function getFormValidationSchema() {
  return yup.object({
    [FN.COMPANY_NAME]: validation.isRequiredString(),
    [FN.COMPANY_ADDRESS]: validation.isRequiredString(),
    [FN.SOME_DATE_FIELD]: validation.isRequiredDate(),
  });
}

```
### 2. Add sample route in `routes/public.route.js` file
```javascript
...
    { path: "/sample", element: SamplePage },
...
```

## Best practices to follow

### 1. Use functional components:
- hooks are available only on functional component
- most of lifecycle hooks provided by class component can be create in functional component and is more easier

### 2. Keys
- all child element must have key attribute while rendering inside loop
- avoid index as key, always use unique property from array as key, if not found generate a unique id using uuid utility function

### 3. Styling
- always use tailwind utility classes for styling
- if using third party plugins and not able to use tailwind only then use plain css and that to by creating the separate css file for the component, don't add css into the global index.cs file directly
- while using tailwind, avoid arbitrary values for classes like text-[16px], bg-[#dedede]
- if any style values like font-size, colors getting used more than 3 times, move it into tailwind.config.js

### 4. While creating custom component place it in the folder as per priority:
- if component is getting use only in one page create it in the page level component folder
- if component is getting use in multiple pages and only depend on the props, create it in global component folder
- if component is getting use in multiple pages and depends on state as well as props, create it in modules folder

### 5. Avoid passing too many attributes/props to the component, use component composition,

e.g.

#### - multiple props

```javascript
<Modal
  heading=""
  body=""
  footer=""
  onHeadingClick={...}
  onFooterClick={...}
  onBodyClick={...}
/>
```

#### - component composition

```javascript
<Modal>
  <Modal.Heading onClick={...}>...</Modal.Heading>
  <Modal.Body onClick={...}>...</Modal.Body>
  <Modal.Footer onClick={...}>...</Modal.Footer>
</Modal>
```

### 6. All strings which are used more than three times should be inside constants
- Constants for repeated String Literals (Const)
- Constants for repeated Array (Enums)

### 7. Form helper

- id, name attributes of form input and htmlFor attribute of form label should be defined at the same place (ideally inside form-helper.js file)
- formik validation schema and initial values should be defined inside form-helper.js files
- No hardcoded form keys (Defined it in form-helper)

### 8. Use context to access data from parent, avoid passing props to child elements

- avoid putting static data inside context, instead export it directly and import it whenever needed.

### 9. Avoid inline event, define the eventHandlers and used its reference inside jsx

### 10. Avoid using js logic in jsx, declare variables and use the variables in jsx, it will make ui more readable

e.g

- using js computations directly in the jsx

```javascript
function component{
  ...
  return (
    <div className="names">
      <div className="short-name">
        {user.first_name[0] + user.last_name[0]}
      </div>
      <div className="full-name">
        {user.first_name + user.last_name}
      </div>
    </div>
  )
}
```

- define and then use variables in the jsx

```javascript
function component{
  ...
  let shortName = user.first_name[0] + user.last_name[0];
  let fullName = user.first_name + user.last_name;
  return (
    <div className="names">
      <div className="short-name">
        {shortName}
      </div>
      <div className="full-name">
        {fullName}
      </div>
    </div>
  )
}
```

### 11. The component should only render jsx, all the js logic (apart from derive variables) should go inside custom hook, and only the variables that are required for views are returned from hook to be used in jsx. No inline functions in jsx.

### 12. Use cx utility function for dynamic className generation, avoid using `background-${isError?"red":"green"}` instead use `cx("background-green",isError && "background-red")`

### 13. Don't commit commented code, console.log and debugger. No Unused/Dead code.

### 14. Handle exceptions appropriately.
- For e.g. if .find() function does not return anything, set a default value for the same since otherwise it returns undefined.

### 14. Use optional chaining while accessing object.
- For e.g. avoid using `object.data.data` instead use `object?.data?.data`

### 15. Variables/Functions/File names nomenclature semantics -
- kebab-case for all folder/file filenames	: file-name.js
- PascalCase for react components 					: ComponentName
- camelCase for everything readFileSync			: variableName 
- No Spelling and Grammar Mistakes



### 17. No state management (useState/useReducer/useEffect ..etc) or function definition on View (jsx) files, move this into provider.

### 18. Convert Complicated Expressions into Variable or Helper Method.

- use `npm run lint` before commenting the code,
- if using vscode, install `tailwind css intellisense' and 'eslint' extension, it will make your life easy.


## PR Checklist

- Ensure that proper naming conventions and casing are followed.
- Avoid using a debugger or console.log in the code.
- Remove any unused or unreachable code from the codebase.
- Do not include any commented-out code in the codebase.
- Do not define variables or functions in JSX or view files.
- Maintain proper formatting and indentation in the code.
- Use "let" and "const" instead of "var", and prefer using "const" when the value does not get modified.
- Avoid using inline functions in view/JSX.
- Reduce code duplication as much as possible.
- Remove any unnecessary code from the component.
- Try to avoid using ternary operators when rendering components conditionally.
- Use optional chaining when required, and provide fallback values if necessary.
- Use constants and enums in the code.
- Submit pull requests for different modules or tasks separately.
- Use formhelpers for forms, including generating payloads, prefilling values, and getting validation schemas.
- Use references only in JSX/view and avoid using logic in HTML; derive variables and use references instead.
