## WordPress Development Coding Conventions

## Table of Contents
- [Test-Driven Development](#test-driven-development)
- [Project Structure](#project-structure)
- [PHP Standards](#php-standards)
- [Security Practices](#security-practices)
- [Block Development](#block-development)
- [JavaScript Standards](#javascript-standards)
- [Database Patterns](#database-patterns)
- [REST API Development](#rest-api-development)
- [Theme Development](#theme-development)

---

## Test-Driven Development

### MANDATORY: Test Before Returning

**🔴 CRITICAL RULE: ALWAYS test your code before marking any feature as complete.**

This is the #1 priority in the development workflow. Testing provides:
- Automatic feedback mechanism for code verification
- Reduced debugging cycles
- Higher code quality
- Faster iteration speed

### Testing Philosophy

When you write tests and give yourself the capability to run those tests, you create a mechanism to automatically feed back to yourself on your own work. This means:

1. **Implement new features** → Run tests → Verify everything works
2. **No manual testing loop** → Tests replace the copy/paste error workflow
3. **Automated iteration** → Tests run automatically, reducing interruptions

### Testing Requirements

#### For WordPress Plugins & Themes:

**Use PHPUnit and WP-CLI test suite:**

```bash
# Install PHPUnit
composer require --dev phpunit/phpunit

# Set up WordPress tests
wp scaffold plugin-tests my-plugin

# Run tests
./vendor/bin/phpunit
```

#### Testing Rules:

1. ✅ **Before marking feature complete** → Run all tests
2. ✅ **Write tests for new functionality** → Unit and integration tests
3. ✅ **Run existing test suite after changes** → Ensure nothing broke
4. ✅ **If formal tests don't exist** → Create temporary test scripts
5. ❌ **NEVER return code without verification** → This wastes time

### When You Can Skip Formal Tests (Prototyping)

For rapid prototyping, you can add this phrase to your workflow:

**"Test this feature before returning"**

This will trigger at least one of these approaches:
- Write ad hoc tests during the run
- Write temporary scripts to test the API
- Manually verify the UI implementation

**Even without formal tests, this phrase makes you iterate harder on your own output before returning.**

### Test-Driven Development Workflow

```
1. Write a failing test
   ↓
2. Implement the feature
   ↓
3. Run the test
   ↓
4. Make the test pass
   ↓
5. Refactor if needed
   ↓
6. Verify all tests still pass
   ↓
7. Commit working code
```

### Example: PHPUnit Test for WordPress

```php
<?php
/**
 * Test custom post type registration
 */
class Test_Custom_Post_Types extends WP_UnitTestCase {
	
	/**
	 * Test that event post type is registered
	 */
	public function test_event_post_type_exists() {
		$this->assertTrue( post_type_exists( 'event' ) );
	}
	
	/**
	 * Test event post type labels
	 */
	public function test_event_post_type_labels() {
		$post_type = get_post_type_object( 'event' );
		$this->assertEquals( 'Events', $post_type->labels->name );
		$this->assertEquals( 'Event', $post_type->labels->singular_name );
	}
	
	/**
	 * Test creating an event post
	 */
	public function test_create_event_post() {
		$post_id = $this->factory->post->create( array(
			'post_type'  => 'event',
			'post_title' => 'Test Event'
		) );
		
		$this->assertGreaterThan( 0, $post_id );
		$this->assertEquals( 'event', get_post_type( $post_id ) );
	}
	
	/**
	 * Test event meta data
	 */
	public function test_event_meta_date() {
		$post_id = $this->factory->post->create( array(
			'post_type' => 'event'
		) );
		
		$event_date = '20251225';
		update_post_meta( $post_id, 'event_date', $event_date );
		
		$retrieved_date = get_post_meta( $post_id, 'event_date', true );
		$this->assertEquals( $event_date, $retrieved_date );
	}
}
```

### Example: JavaScript Test (Jest)

```javascript
import { render, screen, fireEvent } from '@testing-library/react'
import MyComponent from './MyComponent'

describe('MyComponent', () => {
	test('renders component with text', () => {
		render(<MyComponent text="Hello World" />)
		expect(screen.getByText('Hello World')).toBeInTheDocument()
	})
	
	test('button click updates state', () => {
		render(<MyComponent />)
		const button = screen.getByRole('button')
		fireEvent.click(button)
		expect(screen.getByText('Clicked')).toBeInTheDocument()
	})
})
```

### Setting Up Testing Framework

#### PHPUnit Configuration (phpunit.xml)

```xml
<?xml version="1.0"?>
<phpunit
	bootstrap="tests/bootstrap.php"
	backupGlobals="false"
	colors="true"
	convertErrorsToExceptions="true"
	convertNoticesToExceptions="true"
	convertWarningsToExceptions="true"
>
	<testsuites>
		<testsuite name="plugin">
			<directory prefix="test-" suffix=".php">./tests/</directory>
		</testsuite>
	</testsuites>
</phpunit>
```

#### Composer Configuration (composer.json)

```json
{
  "require-dev": {
    "phpunit/phpunit": "^9.5",
    "yoast/phpunit-polyfills": "^1.0"
  },
  "scripts": {
    "test": "phpunit",
    "test:coverage": "phpunit --coverage-html coverage"
  }
}
```

### Pre-Commit Testing with Husky

**Install Husky:**

```bash
npm install --save-dev husky
npx husky install
```

**Add pre-commit hook:**

```bash
npx husky add .husky/pre-commit "npm run test"
```

**.husky/pre-commit:**

```bash
#!/bin/sh
. "$(dirname "$0")/_/husky.sh"

# Run PHP tests
composer test

# Run linting
composer lint

# Run JavaScript tests
npm run test

# Only allow commit if all pass
```

### Testing Commands Reference

```bash
# PHP Tests
composer test              # Run all PHPUnit tests
composer test:unit         # Run unit tests only
composer test:integration  # Run integration tests
composer test:coverage     # Generate coverage report

# JavaScript Tests
npm test                   # Run Jest tests
npm run test:watch        # Run tests in watch mode
npm run test:coverage     # Generate coverage report

# WordPress CLI Tests
wp scaffold plugin-tests my-plugin
cd /tmp/wordpress-tests-lib
bash bin/install-wp-tests.sh wordpress_test root '' localhost latest
```

### What to Test

#### ✅ Always Test:
- Public API functions
- Custom post type registration
- Meta data handling
- Database queries
- REST API endpoints
- User permission checks
- Data sanitization/validation
- Block render output
- JavaScript state management

#### ⚠️ Consider Testing:
- Complex business logic
- Edge cases
- Error handling
- Integration points

#### ❌ Don't Test:
- WordPress core functions
- Third-party library internals
- Simple getters/setters with no logic

### Testing Best Practices

1. **Test behavior, not implementation** → Test what code does, not how
2. **One assertion per test** → Keep tests focused
3. **Use descriptive test names** → `test_user_can_create_event_if_logged_in()`
4. **Arrange, Act, Assert** → Set up → Execute → Verify
5. **Mock external dependencies** → Don't hit real APIs or databases in unit tests
6. **Clean up after tests** → Delete test posts, reset state
7. **Run tests before committing** → Automated via pre-commit hooks

### TDD Statistics

Without tests, typical workflow:
- ⏱️ Write code (10 min)
- ⏱️ Manual test (5 min)
- ⏱️ Find bug (2 min)
- ⏱️ Copy error to AI (1 min)
- ⏱️ Debug (10 min)
- ⏱️ Manual retest (5 min)
- **Total: ~33 minutes**

With tests:
- ⏱️ Write test (3 min)
- ⏱️ Write code (10 min)
- ⏱️ Run test (10 seconds)
- ⏱️ Fix issues (5 min)
- ⏱️ Rerun test (10 seconds)
- **Total: ~18 minutes**

**Result: 45% time savings + higher quality code**

---

## Project Structure

### Block Theme Architecture
```
theme-name/
├── src/                    # Source JS for theme logic
│   ├── modules/           # Shared JS modules
│   └── index.js           # Main entry point
├── our-blocks/            # Custom blocks (legacy)
├── build/                 # Compiled output
│   ├── style-index.css
│   ├── *.js
│   └── *.asset.php
├── templates/             # Block theme templates (FSE)
│   ├── index.html
│   └── single.html
├── template-parts/        # Content templates for CPTs
│   ├── content-program.php
│   └── content-post.php
├── inc/                   # REST API routes + custom PHP
│   ├── search-route.php
│   └── like-route.php
├── css/                   # Styles
├── images/                # Theme images
├── functions.php          # Theme setup
├── style.css             # Theme stylesheet
└── package.json          # Build configuration
```

---

## PHP Standards

### Code Style

**Indentation:**
- Use **TABS** for indentation (not spaces)
- Opening braces on same line for functions/classes
- Always use braces for control structures, even single lines
- Maximum line length: 100 characters (soft limit)

**Example:**
```php
function prefix_my_function( $arg1, $arg2 ) {
	if ( $arg1 > 0 ) {
		return $arg2;
	}
}
```

### Naming Conventions

| Type | Convention | Example |
|------|-----------|---------|
| Functions | `prefix_function_name()` | `university_custom_rest()` |
| Classes | `Prefix_Class_Name` | `JSXBlock`, `PlaceholderBlock` |
| Hooks | `prefix/hook_name` | `university/v1/search` |
| Files | lowercase-with-hyphens | `class-my-class.php` |
| Variables | `$snake_case` | `$our_current_user` |

**CRITICAL: Always prefix all functions, classes, and hooks to avoid conflicts.**

### Documentation (DocBlocks)

All functions MUST have DocBlocks including:
- Description
- `@param` for each parameter
- `@return` for return value
- `@since` for version introduced

**Example:**
```php
/**
 * Retrieves user data from the database.
 *
 * @since 1.0.0
 *
 * @param int    $user_id User ID.
 * @param string $context Context for retrieval.
 * @return array|false User data array or false on failure.
 */
function prefix_get_user_data( $user_id, $context = 'view' ) {
	// Function code
}
```

---

## Security Practices

### Input Sanitization

**ALWAYS sanitize user input:**

```php
// Text fields
$safe_text = sanitize_text_field( $_POST['field_name'] );

// Textarea (no HTML)
$safe_textarea = sanitize_textarea_field( $_POST['textarea'] );

// Email
$safe_email = sanitize_email( $_POST['email'] );

// Integers
$safe_int = absint( $_POST['number'] );

// Keys/slugs
$safe_key = sanitize_key( $_POST['key'] );
```

### Output Escaping

**ALWAYS escape output:**

```php
// HTML content
echo esc_html( $content );

// HTML attributes
echo '<div class="' . esc_attr( $class ) . '">';

// URLs
echo '<a href="' . esc_url( $url ) . '">';

// Allow limited HTML (post content)
echo wp_kses_post( $content );
```

### Nonces and Capabilities

**Always verify permissions:**

```php
// Verify nonce
if ( ! wp_verify_nonce( $_POST['nonce'], 'action_name' ) ) {
	die( 'Security check failed' );
}

// Check capabilities
if ( ! current_user_can( 'edit_posts' ) ) {
	die( 'Insufficient permissions' );
}
```

### Database Safety

**ALWAYS use prepared statements:**

```php
global $wpdb;

// Using prepare()
$results = $wpdb->get_results(
	$wpdb->prepare(
		"SELECT * FROM {$wpdb->prefix}table WHERE column = %s AND id = %d",
		$string_value,
		$int_value
	)
);

// Preferred: Use insert/update methods
$wpdb->insert(
	$wpdb->prefix . 'table',
	array(
		'column1' => $value1,
		'column2' => $value2
	),
	array( '%s', '%d' ) // Format
);
```

**NEVER interpolate user input directly into SQL.**

---

## Block Development

### Modern Block Structure (API v3)

```
src/blockname/
├── block.json       # Metadata
├── index.js         # Registration
├── edit.js          # Editor UI
├── render.php       # Server-side render (optional)
└── style.scss       # Styles (optional)
```

### block.json Template

```json
{
  "$schema": "https://schemas.wp.org/trunk/block.json",
  "apiVersion": 3,
  "name": "namespace/blockname",
  "title": "Block Title",
  "category": "text",
  "attributes": {
    "content": {
      "type": "string",
      "default": ""
    }
  },
  "supports": {
    "align": ["wide", "full"],
    "color": {
      "background": true,
      "text": true
    }
  },
  "editorScript": "file:./index.js",
  "render": "file:./render.php"
}
```

### Block Registration

**index.js:**
```js
import { registerBlockType } from '@wordpress/blocks'
import metadata from './block.json'
import Edit from './edit'

registerBlockType( metadata.name, {
	edit: Edit,
	save: () => null // For dynamic blocks
} )
```

**edit.js:**
```js
import { useBlockProps, RichText } from '@wordpress/block-editor'

export default function Edit( props ) {
	const blockProps = useBlockProps()
	
	return (
		<div { ...blockProps }>
			<RichText
				tagName="p"
				value={ props.attributes.content }
				onChange={ ( content ) => props.setAttributes( { content } ) }
			/>
		</div>
	)
}
```

**CRITICAL: Always use `useBlockProps()` in API v3 blocks.**

### Block Registration in functions.php

```php
function register_custom_blocks() {
	register_block_type_from_metadata( __DIR__ . '/build/blockname' );
}
add_action( 'init', 'register_custom_blocks' );
```

---

## JavaScript Standards

### Code Style

- Use `const` and `let` (not `var`)
- Arrow functions for callbacks
- Destructuring where appropriate
- Async/await for asynchronous operations

**Example:**
```js
import axios from 'axios'

class MyClass {
	constructor() {
		this.events()
	}
	
	events() {
		document.querySelector('.btn').addEventListener('click', () => {
			this.handleClick()
		})
	}
	
	async handleClick() {
		try {
			const response = await axios.get('/api/endpoint')
			console.log(response.data)
		} catch (error) {
			console.error(error)
		}
	}
}

export default MyClass
```

### Module Pattern

**src/index.js:**
```js
import '../css/style.scss'
import MobileMenu from './modules/MobileMenu'
import Search from './modules/Search'

// Instantiate
const mobileMenu = new MobileMenu()
const search = new Search()
```

---

## Database Patterns

### Custom Query Class Pattern

```php
class GetPets {
	public $args;
	public $placeholders;
	public $pets;
	public $count;
	
	function __construct() {
		global $wpdb;
		$tablename = $wpdb->prefix . 'pets';
		
		$this->args = $this->getArgs();
		$this->placeholders = $this->createPlaceholders();
		
		$query = "SELECT * FROM $tablename ";
		$countQuery = "SELECT COUNT(*) FROM $tablename ";
		
		$query .= $this->createWhereText();
		$countQuery .= $this->createWhereText();
		
		$query .= " LIMIT 100";
		
		$this->count = $wpdb->get_var(
			$wpdb->prepare( $countQuery, $this->placeholders )
		);
		
		$this->pets = $wpdb->get_results(
			$wpdb->prepare( $query, $this->placeholders )
		);
	}
	
	private function createWhereText() {
		$where = array();
		
		if ( isset( $this->args['species'] ) ) {
			$where[] = "species = %s";
		}
		
		if ( isset( $this->args['minyear'] ) ) {
			$where[] = "yearofbirth >= %d";
		}
		
		return count( $where ) ? "WHERE " . implode( " AND ", $where ) : "";
	}
	
	private function createPlaceholders() {
		$placeholders = array();
		
		if ( isset( $this->args['species'] ) ) {
			$placeholders[] = $this->args['species'];
		}
		
		if ( isset( $this->args['minyear'] ) ) {
			$placeholders[] = (int) $this->args['minyear'];
		}
		
		return $placeholders;
	}
	
	private function getArgs() {
		$temp = array();
		
		if ( isset( $_GET['species'] ) ) {
			$temp['species'] = sanitize_text_field( $_GET['species'] );
		}
		
		if ( isset( $_GET['minyear'] ) ) {
			$temp['minyear'] = (int) sanitize_text_field( $_GET['minyear'] );
		}
		
		return $temp;
	}
}
```

### Creating Custom Tables

```php
function create_custom_table() {
	global $wpdb;
	$charset = $wpdb->get_charset_collate();
	$tablename = $wpdb->prefix . 'custom_table';
	
	$sql = "CREATE TABLE $tablename (
		id mediumint(9) NOT NULL AUTO_INCREMENT,
		name varchar(60) NOT NULL,
		email varchar(100) NOT NULL,
		created_at datetime DEFAULT CURRENT_TIMESTAMP,
		PRIMARY KEY (id)
	) $charset;";
	
	require_once( ABSPATH . 'wp-admin/includes/upgrade.php' );
	dbDelta( $sql );
}
register_activation_hook( __FILE__, 'create_custom_table' );
```

---

## REST API Development

### Registering Routes

```php
add_action( 'rest_api_init', 'register_custom_routes' );

function register_custom_routes() {
	register_rest_route( 'namespace/v1', '/endpoint', array(
		'methods'             => WP_REST_Server::READABLE,
		'callback'            => 'handle_endpoint',
		'permission_callback' => '__return_true'
	) );
	
	register_rest_route( 'namespace/v1', '/secure-endpoint', array(
		'methods'             => 'POST',
		'callback'            => 'handle_secure_endpoint',
		'permission_callback' => function() {
			return is_user_logged_in();
		}
	) );
}
```

### REST API Handler Example

```php
function handle_endpoint( $data ) {
	$term = sanitize_text_field( $data['term'] );
	
	$query = new WP_Query( array(
		'post_type' => 'post',
		's'         => $term
	) );
	
	$results = array();
	
	while ( $query->have_posts() ) {
		$query->the_post();
		$results[] = array(
			'title'     => get_the_title(),
			'permalink' => get_the_permalink(),
			'excerpt'   => get_the_excerpt()
		);
	}
	
	wp_reset_postdata();
	
	return $results;
}
```

### Adding Custom REST Fields

```php
add_action( 'rest_api_init', 'add_custom_fields' );

function add_custom_fields() {
	register_rest_field( 'post', 'author_name', array(
		'get_callback' => function() {
			return get_the_author();
		}
	) );
}
```

---

## Theme Development

### functions.php Structure

```php
<?php

// Include custom files
require get_theme_file_path( '/inc/custom-routes.php' );

// Enqueue scripts and styles
function theme_enqueue_assets() {
	wp_enqueue_style(
		'theme-styles',
		get_theme_file_uri( '/build/style-index.css' ),
		array(),
		'1.0'
	);
	
	wp_enqueue_script(
		'theme-scripts',
		get_theme_file_uri( '/build/index.js' ),
		array( 'jquery' ),
		'1.0',
		true
	);
	
	// Localize script data
	wp_localize_script( 'theme-scripts', 'themeData', array(
		'root_url' => get_site_url(),
		'nonce'    => wp_create_nonce( 'wp_rest' )
	) );
}
add_action( 'wp_enqueue_scripts', 'theme_enqueue_assets' );

// Theme support
function theme_setup() {
	add_theme_support( 'title-tag' );
	add_theme_support( 'post-thumbnails' );
	add_image_size( 'custom-size', 400, 260, true );
}
add_action( 'after_setup_theme', 'theme_setup' );

// Register blocks
function register_custom_blocks() {
	register_block_type_from_metadata( __DIR__ . '/build/banner' );
	register_block_type_from_metadata( __DIR__ . '/build/header' );
}
add_action( 'init', 'register_custom_blocks' );
```

### Custom Post Type Registration

```php
function register_custom_post_types() {
	register_post_type( 'event', array(
		'public'       => true,
		'show_in_rest' => true,
		'labels'       => array(
			'name'          => 'Events',
			'singular_name' => 'Event'
		),
		'menu_icon'    => 'dashicons-calendar',
		'supports'     => array( 'title', 'editor', 'thumbnail' )
	) );
}
add_action( 'init', 'register_custom_post_types' );
```

### Query Modifications

```php
function modify_queries( $query ) {
	if ( is_admin() || ! $query->is_main_query() ) {
		return;
	}
	
	if ( is_post_type_archive( 'event' ) ) {
		$today = date( 'Ymd' );
		$query->set( 'meta_key', 'event_date' );
		$query->set( 'orderby', 'meta_value_num' );
		$query->set( 'order', 'ASC' );
		$query->set( 'meta_query', array(
			array(
				'key'     => 'event_date',
				'compare' => '>=',
				'value'   => $today,
				'type'    => 'numeric'
			)
		) );
	}
}
add_action( 'pre_get_posts', 'modify_queries' );
```

---

## Build Configuration

### package.json

```json
{
  "name": "theme-name",
  "version": "1.0.0",
  "scripts": {
    "start": "wp-scripts start src/index.js",
    "blocks": "wp-scripts start --experimental-modules",
    "build": "run-s buildIndex buildBlocks",
    "buildIndex": "wp-scripts build src/index.js",
    "buildBlocks": "wp-scripts build --experimental-modules"
  },
  "dependencies": {
    "@wordpress/scripts": "^27.7.0",
    "axios": "^1.6.7",
    "normalize.css": "^8.0.1"
  }
}
```

**Commands:**
- `npm start` - Development build for main theme JS
- `npm run blocks` - Development build for blocks
- `npm run build` - Production build (both)

---

## Best Practices Summary

### PHP
✅ Use tabs for indentation
✅ Prefix all functions, classes, hooks
✅ Always sanitize input
✅ Always escape output
✅ Use prepared statements for database queries
✅ Include comprehensive DocBlocks
✅ Check capabilities and verify nonces

### JavaScript
✅ Use modern ES6+ syntax
✅ Modular architecture with classes
✅ Async/await for API calls
✅ Error handling with try/catch

### Blocks
✅ Use API v3 with `useBlockProps()`
✅ Separate edit.js from index.js
✅ Use block.json for metadata
✅ Dynamic rendering with render.php when needed

### Security
✅ **NEVER** trust user input
✅ **ALWAYS** sanitize and validate
✅ **ALWAYS** escape output
✅ **ALWAYS** check permissions
✅ **ALWAYS** verify nonces

---

## Common WordPress Functions Reference

### Getting Data
```php
get_the_title()
get_the_permalink()
get_the_content()
get_the_excerpt()
get_the_post_thumbnail_url()
get_field() // ACF
get_option()
get_post_meta()
```

### Setting Data
```php
update_option()
update_post_meta()
wp_insert_post()
wp_update_post()
wp_delete_post()
```

### URLs
```php
site_url()
home_url()
get_theme_file_uri()
get_stylesheet_directory_uri()
admin_url()
```

### Security
```php
wp_verify_nonce()
wp_create_nonce()
current_user_can()
is_user_logged_in()
```

---

## Plugin Architecture Pattern

### Constructor-Based Class

```php
class My_Plugin {
	function __construct() {
		add_action( 'admin_menu', array( $this, 'admin_page' ) );
		add_action( 'admin_init', array( $this, 'settings' ) );
	}
	
	function admin_page() {
		add_options_page(
			'Plugin Settings',
			'My Plugin',
			'manage_options',
			'my-plugin',
			array( $this, 'settings_html' )
		);
	}
	
	function settings() {
		register_setting( 'my_plugin', 'my_plugin_option', array(
			'sanitize_callback' => 'sanitize_text_field',
			'default'           => ''
		) );
	}
	
	function settings_html() {
		?>
		<div class="wrap">
			<h1><?php echo esc_html( get_admin_page_title() ); ?></h1>
			<form method="post" action="options.php">
				<?php
				settings_fields( 'my_plugin' );
				do_settings_sections( 'my_plugin' );
				?>
				<input type="text" name="my_plugin_option" 
				       value="<?php echo esc_attr( get_option( 'my_plugin_option' ) ); ?>">
				<?php submit_button(); ?>
			</form>
		</div>
		<?php
	}
}

new My_Plugin();
```

---

**Last Updated:** 2025
**WordPress Version:** 6.0+
**Block API Version:** 3