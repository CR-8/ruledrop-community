Build accessible, themeable React UIs with Chakra UI. Use this skill when composing layouts, implementing dark mode, customizing the design system, or building forms and interactive components with Chakra.

## Setup

```tsx
// main.tsx / app root
import { ChakraProvider } from '@chakra-ui/react'
import { theme } from './theme'

function App() {
  return (
    <ChakraProvider theme={theme}>
      {/* app */}
    </ChakraProvider>
  )
}
```

## Custom Theme

Extend the default theme — don't replace it. Override only what you need:

```ts
// theme/index.ts
import { extendTheme, type ThemeConfig } from '@chakra-ui/react'

const config: ThemeConfig = {
  initialColorMode: 'system',
  useSystemColorMode: true,
}

export const theme = extendTheme({
  config,
  colors: {
    brand: {
      50:  '#eff6ff',
      100: '#dbeafe',
      500: '#3b82f6',
      600: '#2563eb',
      700: '#1d4ed8',
      900: '#1e3a8a',
    },
  },
  fonts: {
    heading: `'Inter', sans-serif`,
    body: `'Inter', sans-serif`,
    mono: `'JetBrains Mono', monospace`,
  },
  fontSizes: {
    xs: '0.75rem',
    sm: '0.875rem',
    md: '1rem',
    lg: '1.125rem',
    xl: '1.25rem',
  },
  radii: {
    sm: '4px',
    md: '6px',
    lg: '8px',
    xl: '12px',
  },
  components: {
    Button: {
      defaultProps: { colorScheme: 'brand' },
      variants: {
        solid: {
          fontWeight: 500,
          _hover: { opacity: 0.9 },
        },
      },
    },
    Input: {
      defaultProps: { focusBorderColor: 'brand.500' },
    },
  },
})
```

## Layout Components

Chakra's layout primitives replace most CSS:

```tsx
import { Box, Flex, Grid, Stack, HStack, VStack, Center, Container } from '@chakra-ui/react'

// Responsive grid
<Grid templateColumns={{ base: '1fr', md: 'repeat(2, 1fr)', lg: 'repeat(3, 1fr)' }} gap={6}>
  {items.map(item => <GridItem key={item.id} />)}
</Grid>

// Flex with gap
<HStack spacing={4} align="center" justify="space-between">
  <Logo />
  <Nav />
</HStack>

// Vertical stack
<VStack spacing={3} align="stretch">
  <Input placeholder="Name" />
  <Input placeholder="Email" />
  <Button type="submit">Submit</Button>
</VStack>
```

## Dark Mode

Use `useColorModeValue` for values that differ between modes:

```tsx
import { useColorModeValue, useColorMode } from '@chakra-ui/react'

function Card({ children }) {
  const bg = useColorModeValue('white', 'gray.800')
  const border = useColorModeValue('gray.200', 'gray.700')
  const text = useColorModeValue('gray.800', 'gray.100')

  return (
    <Box bg={bg} borderColor={border} borderWidth={1} borderRadius="lg" p={4} color={text}>
      {children}
    </Box>
  )
}

function ThemeToggle() {
  const { colorMode, toggleColorMode } = useColorMode()
  return (
    <IconButton
      aria-label="Toggle color mode"
      icon={colorMode === 'dark' ? <SunIcon /> : <MoonIcon />}
      onClick={toggleColorMode}
      variant="ghost"
    />
  )
}
```

## Forms

```tsx
import { FormControl, FormLabel, FormErrorMessage, FormHelperText, Input } from '@chakra-ui/react'
import { useForm } from 'react-hook-form'

function ProfileForm() {
  const { register, handleSubmit, formState: { errors } } = useForm()

  return (
    <form onSubmit={handleSubmit(onSubmit)}>
      <VStack spacing={4}>
        <FormControl isInvalid={!!errors.name} isRequired>
          <FormLabel>Name</FormLabel>
          <Input
            {...register('name', { required: 'Name is required', minLength: { value: 2, message: 'Min 2 characters' } })}
            placeholder="Your name"
          />
          <FormErrorMessage>{errors.name?.message as string}</FormErrorMessage>
        </FormControl>

        <FormControl isInvalid={!!errors.email} isRequired>
          <FormLabel>Email</FormLabel>
          <Input type="email" {...register('email', { required: 'Email is required' })} />
          <FormHelperText>We never share your email.</FormHelperText>
          <FormErrorMessage>{errors.email?.message as string}</FormErrorMessage>
        </FormControl>

        <Button type="submit" width="full">Save</Button>
      </VStack>
    </form>
  )
}
```

## The `as` Prop

Use `as` for semantic HTML without losing Chakra styles:

```tsx
// Renders as <nav> but with Box styles
<Box as="nav" display="flex" gap={4}>
  <Box as="a" href="/" _hover={{ color: 'brand.500' }}>Home</Box>
</Box>

// Button that renders as a link
<Button as="a" href="/signup" variant="solid">Get started</Button>
```

## Responsive Styles

Use the array or object syntax for responsive values:

```tsx
// Array: [base, sm, md, lg, xl]
<Text fontSize={['sm', 'md', 'lg']} fontWeight={[400, 500]}>

// Object: explicit breakpoints
<Box w={{ base: '100%', md: '50%', lg: '33%' }} p={{ base: 4, md: 6 }}>
```

## Accessibility Built-ins

Chakra handles most accessibility automatically:
- `Modal` traps focus and restores it on close
- `Menu` supports keyboard navigation (arrow keys, Enter, Escape)
- `Tooltip` uses `aria-describedby`
- `FormControl` links label to input via `htmlFor`/`id`

Always provide `aria-label` on icon-only buttons:

```tsx
<IconButton aria-label="Close dialog" icon={<CloseIcon />} onClick={onClose} />
```

## Key Rules

- Always use `ChakraProvider` at the root — never instantiate components outside it
- Extend the theme, don't replace it — use `extendTheme`
- Use `useColorModeValue` for all color values that change with the theme
- Prefer Chakra layout components (`Stack`, `Grid`, `Flex`) over raw `Box` with flex/grid props
- Use `as` prop for semantic HTML — don't wrap Chakra components in native elements
- Keep component theme overrides in `theme/index.ts`, not inline
