/**
 * Convert a prompt from the ChatML objects to the format used by Claude.
 * @param {object[]} messages Array of messages
 * @param {boolean} addHumanPrefix Add Human prefix
 * @param {boolean} addAssistantPostfix Add Assistant postfix
 * @param {boolean} withSystemPrompt Build system prompt before "\n\nHuman: "
 * @returns {string} Prompt for Claude
 * @copyright Prompt Conversion script taken from RisuAI by kwaroran (GPLv3).
 */
function convertClaudePrompt(messages, addHumanPrefix, addAssistantPostfix, withSystemPrompt) {
	//Looking for a message index with an assistant role and scaning for "user/Human:" before it.
	const firstAssistantIndex = messages.findIndex(message => message.role === "assistant");
	const hasUser = messages.slice(0, firstAssistantIndex).some(message => message.role === "user" || message.content.includes("Human:"));
	
    let requestPrompt = messages.map((v, i) => {
		// Claude doesn't support message names, so we'll just add them to the message content.
        // Now iside the same cycle.
        if (v.name && v.role !== "system") {
            v.content = `${v.name}: ${v.content}`;
            delete v.name;
        }
		
		let prefix = '';
		//new obj with roles : prefixes.
		const rolePrefixes = {
            "assistant": "\n\nAssistant: ",
            "user": "\n\nHuman: ",
            "system": v.name === "example_assistant" ? "\n\nA: " : v.name === "example_user" ? "\n\nH: " : "\n\n"
		}	
	    //For Claude 2.1 adds "/n/n" as prefix, when role of the 1st msg. is 'system' or 'assistant'. Else adds "Human" prefix, when the role is user(for compability with 2.0 format).		
        if (i === 0 && ["system", "assistant"].includes(v.role) && withSystemPrompt) {
            prefix = "\n\n";
		  // If there is no message with role "user" or prefix "Human: ", change the assistant's prefix, inserting the human's massage.
        } else if (i === firstAssistantIndex && !hasUser && withSystemPrompt) {
            prefix = "\n\nHuman: Let's get started.\n\nAssistant: ";
		  // For Claude 2.0- adds the "Human" prefix to the 1st message.
		} else if (i === 0 && addHumanPrefix) {
            prefix = "\n\nHuman: ";
        } else {
			//Sets the correct prefix according to the role.
			prefix = rolePrefixes[v.role] || '\n\n';
        }
        return prefix + v.content;
    }).join('');
	
	/**
    if (withSystemPrompt && !hasUser && firstAssistantIndex > -1) {
        // Another way to insert a missing human. If there is no message with role "user" or prefix "Human: ", add a new message before the first message of the assistant. Remember to comment or remove .join('') above before use.
        requestPrompt.splice(firstAssistantIndex, 0, "\n\nHuman: Let's get started.");
    }
    requestPrompt = requestPrompt.join('');
	*/
    if (addAssistantPostfix) {
        requestPrompt += "\n\nAssistant: ";
    }

    return requestPrompt;
}

module.exports = {
    convertClaudePrompt,
}
