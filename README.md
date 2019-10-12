# Panduan Airbnb React/JSX _Style_

*Pendekatan yang sebagian besar _reasonable_ untuk React dan JSX*

Panduan _style guide_ ini sebagian besar didasarkan pada standar yang saat ini lazim di Javascript, meskipun beberapa konvensi (yaitu async/await atau static class fields) masih bisa dimasukkan atau dilarang berdasarkan kasus per kasus. Saat ini, apapun sebelum tahap ke 3 tidak termasuk atau direkomendasikan dalam panduan ini.

## Daftar Isi

  1. [Aturan Dasar](#aturan-dasar)
  1. [Class vs `React.createClass` vs stateless](#class-vs-reactcreateclass-vs-stateless)
  1. [Mixins](#mixins)
  1. [Penamaan](#penamaan)
  1. [Deklarasi](#deklarasi)
  1. [Alignment](#alignment)
  1. [Quotes](#quotes)
  1. [Spacing](#spacing)
  1. [Props](#props)
  1. [Refs](#refs)
  1. [Tanda Kurung](#tanda-kurung)
  1. [Tags](#tags)
  1. [Methods](#methods)
  1. [Ordering](#ordering)
  1. [`isMounted`](#ismounted)

## Aturan Dasar

  - Hanya menyertakan satu React component per file.
    - Namun, multiple [Stateless, atau Pure, Components](https://facebook.github.io/react/docs/reusable-components.html#stateless-functions) diizinkan per file. eslint: [`react/no-multi-comp`](https://github.com/yannickcr/eslint-plugin-react/blob/master/docs/rules/no-multi-comp.md#ignorestateless).
  - Selalu gunakan sintaks JSX.
  - Jangan gunakan `React.createElement` kecuali Anda menginisialisasi aplikasi dari file yang bukan JSX.
  - [`react/forbid-prop-types`](https://github.com/yannickcr/eslint-plugin-react/blob/master/docs/rules/forbid-prop-types.md) akan memperbolehkan `arrays` dan `objects` hanya jika secara ekplisit ditulis dengan isi `array` dan `object`, menggunakan `arrayOf`, `objectOf`, atau `shape`.

## Class vs `React.createClass` vs stateless

  - Jika Anda memiliki _internal state_ dan/atau _refs_, lebih baik gunakan `class extends React.Component` daripada `React.createClass`. eslint: [`react/prefer-es6-class`](https://github.com/yannickcr/eslint-plugin-react/blob/master/docs/rules/prefer-es6-class.md) [`react/prefer-stateless-function`](https://github.com/yannickcr/eslint-plugin-react/blob/master/docs/rules/prefer-stateless-function.md)

    ```jsx
    // buruk
    const Listing = React.createClass({
      // ...
      render() {
        return <div>{this.state.hello}</div>;
      }
    });

    // baik
    class Listing extends React.Component {
      // ...
      render() {
        return <div>{this.state.hello}</div>;
      }
    }
    ```

    Dan jika Anda tidak memiliki _state_ atau _refs_, lebih baik gunakan fungsi biasa (bukan _arrow functions_) daripada _classes_:

    ```jsx
    // buruk
    class Listing extends React.Component {
      render() {
        return <div>{this.props.hello}</div>;
      }
    }

    // buruk (mengandalkan inferensi nama fungsi yang tidak disarankan)
    const Listing = ({ hello }) => (
      <div>{hello}</div>
    );

    // baik
    function Listing({ hello }) {
      return <div>{hello}</div>;
    }
    ```

## Mixins

  - [Jangan gunakan _mixins_](https://facebook.github.io/react/blog/2016/07/13/mixins-considered-harmful.html).

  > Mengapa? _Mixins_ memperkenalkan dependensi yang implisit, menyebabkan nama tidak sesuai, dan kompleksitas _snowballing_. Sebagian besar kasus untuk _mixins_ dapat diselesaikan dengan cara yang lebih baik melalui _components_, _higher-order components_, atau _utility modules_.

## Penamaan

  - **Ekstensi**: Gunakan ekstensi `.jsx` untuk React components. eslint: [`react/jsx-filename-extension`](https://github.com/yannickcr/eslint-plugin-react/blob/master/docs/rules/jsx-filename-extension.md)
  - **Nama file**: Gunanakan PascalCase untuk nama file. Contoh:  `ReservationCard.jsx`.
  - **Penamaan Referensi**: Gunakan PascalCase untuk React components dan camelCase untuk _intances_ nya. eslint: [`react/jsx-pascal-case`](https://github.com/yannickcr/eslint-plugin-react/blob/master/docs/rules/jsx-pascal-case.md)

    ```jsx
    // buruk
    import reservationCard from './ReservationCard';

    // baik
    import ReservationCard from './ReservationCard';

    // buruk
    const ReservationItem = <ReservationCard />;

    // baik
    const reservationItem = <ReservationCard />;
    ```

  - **Penamaan Komponen**: Gunakan nama file sebagai nama komponen. Sebagai contoh, `ReservationCard.jsx` harus memiliki nama referensi dari `ReservationCard`. Namun, untuk komponen root dari direktori, menggunakan `index.jsx` sebagai nama file  dan gunakan nama direktori sebagai nama komponen:

    ```jsx
    // buruk
    import Footer from './Footer/Footer';

    // buruk
    import Footer from './Footer/index';

    // baik
    import Footer from './Footer';
    ```
  - **Penamaan _Higher-order Component_**: Gunakan sebuah komposit dari nama _higher-order component_ dan nama komponen yang diteruskan sebagai `displayName` pada komponen yang dihasilkan. Sebagai contoh, `withFoo()` pada _higher-order component_ , ketika melewati komponen  `Bar` harus menghasilkan komponen dengan `displayName` dari `withFoo(Bar)`.

    > Mengapa? `displayName` dapat digunakan oleh _developer tools_ atau dalam pesan error, dan memiliki nilai yang secara jelas mengungkapkan hubungan untuk membantu orang memahami apa yang terjadi.

    ```jsx
    // buruk
    export default function withFoo(WrappedComponent) {
      return function WithFoo(props) {
        return <WrappedComponent {...props} foo />;
      }
    }

    // baik
    export default function withFoo(WrappedComponent) {
      function WithFoo(props) {
        return <WrappedComponent {...props} foo />;
      }

      const wrappedComponentName = WrappedComponent.displayName
        || WrappedComponent.name
        || 'Component';

      WithFoo.displayName = `withFoo(${wrappedComponentName})`;
      return WithFoo;
    }
    ```

  - **Penamaan Props**: Hindari penggunaan nama prop pada komponen DOM untuk tujuan yang berbeda.

    > Mengapa? Orang akan berpikir props seperti `style` dan `className` berarti satu hal tertentu. Memvariasikan API ini untuk sebagian dari aplikasi Anda sehingga membuat kode lebih mudah dibaca dan kurang terpelihara, dan dapat menyebabkan bug.

    ```jsx
    // buruk
    <MyComponent style="fancy" />

    // buruk
    <MyComponent className="fancy" />

    // baik
    <MyComponent variant="fancy" />
    ```

## Deklarasi

  - Jangan gunakan `displayName` untuk penamaai sebuah komponen. Sebagai gantinya, beri nama pada komponen dengan referensi.

    ```jsx
    // buruk
    export default React.createClass({
      displayName: 'ReservationCard',
      // stuff goes here
    });

    // baik
    export default class ReservationCard extends React.Component {
    }
    ```

## Alignment

  - Ikuti _alignment styles_ ini untuk sintaks JSX. eslint: [`react/jsx-closing-bracket-location`](https://github.com/yannickcr/eslint-plugin-react/blob/master/docs/rules/jsx-closing-bracket-location.md) [`react/jsx-closing-tag-location`](https://github.com/yannickcr/eslint-plugin-react/blob/master/docs/rules/jsx-closing-tag-location.md)

    ```jsx
    // buruk
    <Foo superLongParam="bar"
         anotherSuperLongParam="baz" />

    // baik
    <Foo
      superLongParam="bar"
      anotherSuperLongParam="baz"
    />

    // jika properti dimuat dalam satu baris maka tetap pada satu baris yang sama
    <Foo bar="bar" />

    // children mendapatkan indentasi secara normal
    <Foo
      superLongParam="bar"
      anotherSuperLongParam="baz"
    >
      <Quux />
    </Foo>

    // buruk
    {showButton &&
      <Button />
    }

    // buruk
    {
      showButton &&
        <Button />
    }

    // baik
    {showButton && (
      <Button />
    )}

    // baik
    {showButton && <Button />}
    ```

## Quotes

  - Selalu gunakan tanda kutip ganda (`"`) untuk atribut JSX, tetapi tanda kutip tunggal (`'`) untuk semua JS lainnya. eslint: [`jsx-quotes`](https://eslint.org/docs/rules/jsx-quotes)

    > Mengapa? Atribut HTML juga biasanya menggunakan tanda kutip ganda bukan tunggal, jadi atribut JSX menggunakan ketentuan ini.

    ```jsx
    // buruk
    <Foo bar='bar' />

    // baik
    <Foo bar="bar" />

    // buruk
    <Foo style={{ left: "20px" }} />

    // baik
    <Foo style={{ left: '20px' }} />
    ```

## Spacing

  - Selalu sisipkan satu spasi di tag _self-closing_. eslint: [`no-multi-spaces`](https://eslint.org/docs/rules/no-multi-spaces), [`react/jsx-tag-spacing`](https://github.com/yannickcr/eslint-plugin-react/blob/master/docs/rules/jsx-tag-spacing.md)

    ```jsx
    // buruk
    <Foo/>

    // sangat buruk
    <Foo                 />

    // buruk
    <Foo
     />

    // baik
    <Foo />
    ```

  - Jangan menyisipkan spasi didalam kurung kurawal JSX. eslint: [`react/jsx-curly-spacing`](https://github.com/yannickcr/eslint-plugin-react/blob/master/docs/rules/jsx-curly-spacing.md)

    ```jsx
    // buruk
    <Foo bar={ baz } />

    // baik
    <Foo bar={baz} />
    ```

## Props

  - Selalu gunakan camelCase untuk nama prop.

    ```jsx
    // buruk
    <Foo
      UserName="hello"
      phone_number={12345678}
    />

    // baik
    <Foo
      userName="hello"
      phoneNumber={12345678}
    />
    ```

  - Hilangkan nama prop jika itu secara ekplisit bernilai  `true`. eslint: [`react/jsx-boolean-value`](https://github.com/yannickcr/eslint-plugin-react/blob/master/docs/rules/jsx-boolean-value.md)

    ```jsx
    // buruk
    <Foo
      hidden={true}
    />

    // baik
    <Foo
      hidden
    />

    // baik
    <Foo hidden />
    ```

  - Selalu sertakan `alt` prop pada setiap tag `<img>`. Jika gambar adalah _presentational_, `alt` dapat berisi string kosong atau `<img>` harus memiliki `role="presentation"`. eslint: [`jsx-a11y/alt-text`](https://github.com/evcohen/eslint-plugin-jsx-a11y/blob/master/docs/rules/alt-text.md)

    ```jsx
    // buruk
    <img src="hello.jpg" />

    // baik
    <img src="hello.jpg" alt="Me waving hello" />

    // baik
    <img src="hello.jpg" alt="" />

    // baik
    <img src="hello.jpg" role="presentation" />
    ```

  - Jangan gunakan kata-kata seperti "image", "photo", atau "picture" di `<img>` `alt` _props_. eslint: [`jsx-a11y/img-redundant-alt`](https://github.com/evcohen/eslint-plugin-jsx-a11y/blob/master/docs/rules/img-redundant-alt.md)

    > Mengapa? Screenreaders sudah memberitahu elemen `img` sebagai gambar, jadi tidak perlu memberikan informasi ini didalam alt.

    ```jsx
    // buruk
    <img src="hello.jpg" alt="Picture of me waving hello" />

    // baik
    <img src="hello.jpg" alt="Me waving hello" />
    ```

  - Gunakan yang sesuai, _non-abstract_ [ARIA roles](https://www.w3.org/TR/wai-aria/#usage_intro). eslint: [`jsx-a11y/aria-role`](https://github.com/evcohen/eslint-plugin-jsx-a11y/blob/master/docs/rules/aria-role.md)

    ```jsx
    // buruk - not an ARIA role
    <div role="datepicker" />

    // buruk - abstract ARIA role
    <div role="range" />

    // baik
    <div role="button" />
    ```

  - Jangan gunakan `accessKey` pada elemen. eslint: [`jsx-a11y/no-access-key`](https://github.com/evcohen/eslint-plugin-jsx-a11y/blob/master/docs/rules/no-access-key.md)

  > Mengapa? Ketidakkonsistenan antara _keyboard shortcuts_ dan _keyboard commands_ yang digunakan oleh seseorang yang menggunakan screenreaders dan keyboards mempersulit aksesbilitas.

  ```jsx
  // buruk
  <div accessKey="h" />

  // baik
  <div />
  ```

  - Hindari menggunakan indeks array sebagai `key` pada prop, disarankan menggunakan ID yang lebih stabil. eslint: [`react/no-array-index-key`](https://github.com/yannickcr/eslint-plugin-react/blob/master/docs/rules/no-array-index-key.md)

> Mengapa? Tidak menggunakan ID adalah [_anti-pattern_](https://medium.com/@robinpokorny/index-as-a-key-is-an-anti-pattern-e0349aece318) karena dapat berdampak negatif pada kinerja dan menyebabkan masalah pada _component state_.

Kami tidak merekomendasikan untuk menggunakan indeks sebagai _keys_ jika urutan item dapat berubah.

  ```jsx
  // buruk
  {todos.map((todo, index) =>
    <Todo
      {...todo}
      key={index}
    />
  )}

  // baik
  {todos.map(todo => (
    <Todo
      {...todo}
      key={todo.id}
    />
  ))}
  ```

  - Selalu definisikan defaultProps secara ekplisit untuk semua prop yang tidak diperlukan.

  > Mengapa? propTypes adalah bentuk dokumentasi, dan menyediakan defaultProps yang berarti pembaca kode Anda tidak perlu berasumsi sebanyak itu.

  ```jsx
  // buruk
  function SFC({ foo, bar, children }) {
    return <div>{foo}{bar}{children}</div>;
  }
  SFC.propTypes = {
    foo: PropTypes.number.isRequired,
    bar: PropTypes.string,
    children: PropTypes.node,
  };

  // baik
  function SFC({ foo, bar, children }) {
    return <div>{foo}{bar}{children}</div>;
  }
  SFC.propTypes = {
    foo: PropTypes.number.isRequired,
    bar: PropTypes.string,
    children: PropTypes.node,
  };
  SFC.defaultProps = {
    bar: '',
    children: null,
  };
  ```

  - Gunakan _spread prop_ dengan hemat (_sparingly_).
  > Mengapa? kalau tidak, Anda akan lebih cenderung meneruskan properti yang tidak perlu ke komponen. Dan untuk React v15.6.1 keatas, Anda dapat [mem-passing atribut HTML yang tidak valid ke DOM](https://reactjs.org/blog/2017/09/08/dom-attributes-in-react-16.html).

  Pengecualian:

  - _HOCs that proxy down props_ dan _hoist propTypes_

  ```jsx
  function HOC(WrappedComponent) {
    return class Proxy extends React.Component {
      Proxy.propTypes = {
        text: PropTypes.string,
        isLoading: PropTypes.bool
      };

      render() {
        return <WrappedComponent {...this.props} />
      }
    }
  }
  ```

  - _Spreading_ dengan objek yang dikenal, _explicit props_. Ini bisa sangat bergunakan ketika men-_testing_ komponen React dengan _Mocha's beforeEach construct_.

  ```jsx
  export default function Foo {
    const props = {
      text: '',
      isPublished: false
    }

    return (<div {...props} />);
  }
  ```

  Catatan untuk digunakan:
  Filter prop yang tidak diperlukan jika memungkinkan. Selain itu, gunakan [prop-types-exact](https://www.npmjs.com/package/prop-types-exact) untuk membantu mencegah bug.

  ```jsx
  // buruk
  render() {
    const { irrelevantProp, ...relevantProps } = this.props;
    return <WrappedComponent {...this.props} />
  }

  // baik
  render() {
    const { irrelevantProp, ...relevantProps } = this.props;
    return <WrappedComponent {...relevantProps} />
  }
  ```

## Refs

  - Selalu gunakan _ref callbacks_. eslint: [`react/no-string-refs`](https://github.com/yannickcr/eslint-plugin-react/blob/master/docs/rules/no-string-refs.md)

    ```jsx
    // buruk
    <Foo
      ref="myRef"
    />

    // baik
    <Foo
      ref={(ref) => { this.myRef = ref; }}
    />
    ```

## Tanda Kurung

  - Bungkus tag JSX dengan tanda kurung ketika lebih dari satu baris. eslint: [`react/jsx-wrap-multilines`](https://github.com/yannickcr/eslint-plugin-react/blob/master/docs/rules/jsx-wrap-multilines.md)

    ```jsx
    // buruk
    render() {
      return <MyComponent variant="long body" foo="bar">
               <MyChild />
             </MyComponent>;
    }

    // baik
    render() {
      return (
        <MyComponent variant="long body" foo="bar">
          <MyChild />
        </MyComponent>
      );
    }

    // baik, ketika statement hanya 1 baris
    render() {
      const body = <div>hello</div>;
      return <MyComponent>{body}</MyComponent>;
    }
    ```

## Tags

  - Selalu _self-close tags_ yang tidak memiliki _children_. eslint: [`react/self-closing-comp`](https://github.com/yannickcr/eslint-plugin-react/blob/master/docs/rules/self-closing-comp.md)

    ```jsx
    // buruk
    <Foo variant="stuff"></Foo>

    // baik
    <Foo variant="stuff" />
    ```

  - Jika komponen Anda memilki properti yang _multi-line_, tutup tag-nya di baris baru. eslint: [`react/jsx-closing-bracket-location`](https://github.com/yannickcr/eslint-plugin-react/blob/master/docs/rules/jsx-closing-bracket-location.md)

    ```jsx
    // buruk
    <Foo
      bar="bar"
      baz="baz" />

    // baik
    <Foo
      bar="bar"
      baz="baz"
    />
    ```

## Methods

  - Gunakan _arrow functions_ untuk menutup variabel lokal. Ini berguna ketika Anda memberikan data tambahan ke _event handler_. Meskipun, pastikan mereka [tidak secara masiv merusak peforma](https://www.bignerdranch.com/blog/choosing-the-best-approach-for-react-event-handlers/), khususnya ketika mem-_passing_ ke komponen kustom yang mungkin meruapakan PureComponents, karena mereka akan men-_trigger_ _rerender_ yang mungkin tidak setiap waktu.

    ```jsx
    function ItemList(props) {
      return (
        <ul>
          {props.items.map((item, index) => (
            <Item
              key={item.key}
              onClick={(event) => { doSomethingWith(event, item.name, index); }}
            />
          ))}
        </ul>
      );
    }
    ```

  - _Bind event handlers_ untuk _render method_ di _constructor_. eslint: [`react/jsx-no-bind`](https://github.com/yannickcr/eslint-plugin-react/blob/master/docs/rules/jsx-no-bind.md)

    > Mengapa? sebuah _bind call_ di _render path_ membuat fungsi baru disetiap _single render_. Jangan gunakan _arrow functions_ di _class fields_, karena itu membuat mereka  [_challenging_ untuk _test_ dan men-debug, dan dapat berdampak negatif pada _performance_](https://medium.com/@charpeni/arrow-functions-in-class-properties-might-not-be-as-great-as-we-think-3b3551c440b1), dan karena konseptual, _class fields_ adalah untuk data. bukan logika.

    ```jsx
    // buruk
    class extends React.Component {
      onClickDiv() {
        // lakukan stuff
      }

      render() {
        return <div onClick={this.onClickDiv.bind(this)} />;
      }
    }

    // sangat buruk
    class extends React.Component {
      onClickDiv = () => {
        // lakukan stuff
      }

      render() {
        return <div onClick={this.onClickDiv} />
      }
    }

    // baik
    class extends React.Component {
      constructor(props) {
        super(props);

        this.onClickDiv = this.onClickDiv.bind(this);
      }

      onClickDiv() {
        // do stuff
      }

      render() {
        return <div onClick={this.onClickDiv} />;
      }
    }
    ```

  - Jangan gunakan awalan (_prefix_) untuk _internal methods_ komponen pada React.
    > Mengapa? Awalan garis bawah terkadang digunakan sebagai _convention_ dalam bahasa lain untuk menunjukan _private_. Tapi, tidak seperti bahasa-bahasa itu, tidak ada dukungan asli untuk _private_ di JavaScipt, semuanya bersifat _public_. Terlepas dari niat Anda, menambahkan awalan garis bawah ke properti Anda sebenarnya tidak menjadikan bersifat _private_, dan properti apapun (_underscore-prefixed_ atau bukan) harus diperlakukan secara _public_. Lihat _issues_ [#1024](https://github.com/airbnb/javascript/issues/1024), dan [#490](https://github.com/airbnb/javascript/issues/490) untuk diskusi yang lebih mendalam.

    ```jsx
    // buruk
    React.createClass({
      _onClickSubmit() {
        // lakukan stuff
      },

      // stuff lain
    });

    // baik
    class extends React.Component {
      onClickSubmit() {
        // lakukan stuff
      }

      // stuff lain
    }
    ```

  - Pastikan untuk mengembalikan nilai dalam method render  `render` Anda. eslint: [`react/require-render-return`](https://github.com/yannickcr/eslint-plugin-react/blob/master/docs/rules/require-render-return.md)

    ```jsx
    // buruk
    render() {
      (<div />);
    }

    // baik
    render() {
      return (<div />);
    }
    ```

## Ordering

  - Ordering for `class extends React.Component`:

  1. metode `static` opsional
  1. `constructor`
  1. `getChildContext`
  1. `componentWillMount`
  1. `componentDidMount`
  1. `componentWillReceiveProps`
  1. `shouldComponentUpdate`
  1. `componentWillUpdate`
  1. `componentDidUpdate`
  1. `componentWillUnmount`
  1. *clickHandlers atau eventHandlers* seperti `onClickSubmit()` atau `onChangeDescription()`
  1. *_getter methods_ untuk `render`* seperti `getSelectReason()` atau `getFooterContent()`
  1. *optional render methods* seperti `renderNavigation()` atau `renderProfilePicture()`
  1. `render`

  - How to define `propTypes`, `defaultProps`, `contextTypes`, etc...

    ```jsx
    import React from 'react';
    import PropTypes from 'prop-types';

    const propTypes = {
      id: PropTypes.number.isRequired,
      url: PropTypes.string.isRequired,
      text: PropTypes.string,
    };

    const defaultProps = {
      text: 'Hello World',
    };

    class Link extends React.Component {
      static methodsAreOk() {
        return true;
      }

      render() {
        return <a href={this.props.url} data-id={this.props.id}>{this.props.text}</a>;
      }
    }

    Link.propTypes = propTypes;
    Link.defaultProps = defaultProps;

    export default Link;
    ```

  - Ordering untuk `React.createClass`: eslint: [`react/sort-comp`](https://github.com/yannickcr/eslint-plugin-react/blob/master/docs/rules/sort-comp.md)

  1. `displayName`
  1. `propTypes`
  1. `contextTypes`
  1. `childContextTypes`
  1. `mixins`
  1. `statics`
  1. `defaultProps`
  1. `getDefaultProps`
  1. `getInitialState`
  1. `getChildContext`
  1. `componentWillMount`
  1. `componentDidMount`
  1. `componentWillReceiveProps`
  1. `shouldComponentUpdate`
  1. `componentWillUpdate`
  1. `componentDidUpdate`
  1. `componentWillUnmount`
  1. *clickHandlers or eventHandlers* seperti `onClickSubmit()` atau `onChangeDescription()`
  1. *getter methods for `render`* seperti `getSelectReason()` atau `getFooterContent()`
  1. *optional render methods* seperti `renderNavigation()` atau `renderProfilePicture()`
  1. `render`

## `isMounted`

  - Jangan gunakan `isMounted`. eslint: [`react/no-is-mounted`](https://github.com/yannickcr/eslint-plugin-react/blob/master/docs/rules/no-is-mounted.md)

  > Mengapa? [`isMounted` _is an anti-pattern_][anti-pattern], tidak tersedia saat menggunakan _ES6 classes_, dan sedang dalam perjalanan untuk secara resmi ditinggalkan.

  [anti-pattern]: https://facebook.github.io/react/blog/2015/12/16/ismounted-antipattern.html

## Terjemahan

  Panduan JSX/React style ini juga tersedia dalam bahasa lain:

  - ![cn](https://raw.githubusercontent.com/gosquared/flags/master/flags/flags/shiny/24/China.png) **Chinese (Simplified)**: [JasonBoy/javascript](https://github.com/JasonBoy/javascript/tree/master/react)
  - ![tw](https://raw.githubusercontent.com/gosquared/flags/master/flags/flags/shiny/24/Taiwan.png) **Chinese (Traditional)**: [jigsawye/javascript](https://github.com/jigsawye/javascript/tree/master/react)
  - ![es](https://raw.githubusercontent.com/gosquared/flags/master/flags/flags/shiny/24/Spain.png) **Español**: [agrcrobles/javascript](https://github.com/agrcrobles/javascript/tree/master/react)
  - ![jp](https://raw.githubusercontent.com/gosquared/flags/master/flags/flags/shiny/24/Japan.png) **Japanese**: [mitsuruog/javascript-style-guide](https://github.com/mitsuruog/javascript-style-guide/tree/master/react)
  - ![kr](https://raw.githubusercontent.com/gosquared/flags/master/flags/flags/shiny/24/South-Korea.png) **Korean**: [apple77y/javascript](https://github.com/apple77y/javascript/tree/master/react)
  - ![pl](https://raw.githubusercontent.com/gosquared/flags/master/flags/flags/shiny/24/Poland.png) **Polish**: [pietraszekl/javascript](https://github.com/pietraszekl/javascript/tree/master/react)
  - ![Br](https://raw.githubusercontent.com/gosquared/flags/master/flags/flags/shiny/24/Brazil.png) **Portuguese**: [ronal2do/javascript](https://github.com/ronal2do/airbnb-react-styleguide)
  - ![ru](https://raw.githubusercontent.com/gosquared/flags/master/flags/flags/shiny/24/Russia.png) **Russian**: [leonidlebedev/javascript-airbnb](https://github.com/leonidlebedev/javascript-airbnb/tree/master/react)
  - ![th](https://raw.githubusercontent.com/gosquared/flags/master/flags/flags/shiny/24/Thailand.png) **Thai**: [lvarayut/javascript-style-guide](https://github.com/lvarayut/javascript-style-guide/tree/master/react)
  - ![tr](https://raw.githubusercontent.com/gosquared/flags/master/flags/flags/shiny/24/Turkey.png) **Turkish**: [alioguzhan/react-style-guide](https://github.com/alioguzhan/react-style-guide)
  - ![ua](https://raw.githubusercontent.com/gosquared/flags/master/flags/flags/shiny/24/Ukraine.png) **Ukrainian**: [ivanzusko/javascript](https://github.com/ivanzusko/javascript/tree/master/react)
  - ![vn](https://raw.githubusercontent.com/gosquared/flags/master/flags/flags/shiny/24/Vietnam.png) **Vietnam**: [uetcodecamp/jsx-style-guide](https://github.com/UETCodeCamp/jsx-style-guide)

**[⬆ kembali ke atas](#daftar-isi)**
