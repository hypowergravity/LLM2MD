# LLM2MD
Javascript code to convert LLM generated text in html to markdown


~~~javascript
(() => {
  // 1. Helper function to convert HTML in messages to Markdown
  // it replaces the html tags to markdown format.
  function h(html) {
    return html
      // <p> => double new lines
      .replace(/<p>/g, '\n\n')
      .replace(/<\/p>/g, '')
      // Bold/italics
      .replace(/<b>/g, '**')
      .replace(/<\/b>/g, '**')
      .replace(/<i>/g, '_')
      .replace(/<\/i>/g, '_')
      // Convert <code> blocks into fenced code
      .replace(/<code[^>]*>/g, (match) => {
        const languageMatch = match.match(/class="[^"]*language-([^"]*)"/);
        // If we detect a language (e.g., language-javascript), use ```javascript
        return languageMatch ? `\n\`\`\`${languageMatch[1]}\n` : '\n```\n';
      })
      .replace(/<\/code[^>]*>/g, '\n```\n')
      // Remove all other HTML tags
      .replace(/<[^>]+>/g, '')
      // Remove "Copy code" text (if present)
      .replace(/Copy code/g, '')
      // Remove policy disclaimers (if present)
      .replace(
        /This content may violate our content policy.*research in this area\./g,
        ''
      )
      // Trim extra spaces/newlines
      .trim();
  }

  // 2. Collect all messages (both user and ChatGPT) 
  // by targeting the outer ".text-base" containers
  const messageNodes = document.querySelectorAll('.text-base');

  let mdContent = '';

  for (const node of messageNodes) {
    // Determine who the speaker is.
    // If there's an <img> (avatar), we assume it's the user, otherwise ChatGPT.
    const speaker = node.querySelector('img') ? 'You' : 'ChatGPT';

    // ChatGPT answers are often in .markdown or .prose
    // User messages are often in .whitespace-pre-wrap
    const textContainer =
      node.querySelector('.whitespace-pre-wrap') ||
      node.querySelector('.markdown') ||
      node.querySelector('.prose');

    if (textContainer) {
      // Convert HTML to Markdown
      mdContent += `**${speaker}**: ${h(textContainer.innerHTML)}\n\n`;
    }
  }

  // 3. Create a hidden link, download as Markdown
  const downloadLink = document.createElement('a');
  downloadLink.download = 'Conversation with ChatGPT.md';
  downloadLink.href = URL.createObjectURL(new Blob([mdContent]));
  downloadLink.style.display = 'none';
  document.body.appendChild(downloadLink);
  downloadLink.click();
  downloadLink.remove();
})();

~~~
