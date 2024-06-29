<!--yml

category: 未分类

date: 2024-05-27 14:39:20

-->

# Panda CSS - 利用构建时间和类型安全的 CSS-in-JS 构建现代网站

> 来源：[https://panda-css.com/](https://panda-css.com/)

```
 import { css } from "./styled-system/css"; 

import { circle, stack } from "./styled-system/patterns"; 

function App() { 

 return ( 

 <div className={stack({ direction: "row", p: "4" })}> 

 <div className={circle({ size: "5rem", overflow: "hidden" })}> 

 <img src="https://via.placeholder.com/150" alt="avatar" /> 

 </div> 

 <div className={css({ mt: "4", fontSize: "xl", fontWeight: "semibold" })}> 

 John Doe 

 </div> 

 <div className={css({ mt: "2", fontSize: "sm", color: "gray.600" })}> 

 john@doe.com 

 </div> 

 </div> 

 ); 

} 
```

```
 import { Box, Stack, Circle } from './styled-system/jsx' 

function App() { 

 return ( 

 <Stack direction="row" p="4" rounded="md" shadow="lg" bg="white"> 

 <Circle size="5rem" overflow="hidden"> 

 <img src="https://via.placeholder.com/150" alt="avatar" /> 

 </Circle> 

 <Box mt="4" fontSize="xl" fontWeight="semibold"> 

 John Doe 

 </Box> 

 <Box mt="2" fontSize="sm" color="gray.600"> 

 john@doe.com 

 </Box> 

 </Stack> 

 ) 

} 
```

```
 import { styled } from './styled-system/jsx' 

import { cva } from './styled-system/css' 

export const badge = cva({ 

 base: { 

 fontWeight: 'medium', 

 borderRadius: 'md', 

 }, 

 variants: { 

 status: { 

 default: { 

 color: 'white', 

 bg: 'gray.500', 

 }, 

 success: { 

 color: 'white', 

 bg: 'green.500', 

 }, 

 }, 

 } 

}) 

export const Badge = styled('span', badge) 
```
