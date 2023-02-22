---
title: 'Fetch Discord Upvote Data'
summary: 'This function retrieves the number of upvotes a Discord member has received in the past 24 hours.'
date: '2023-02-22'
author:
  name: Sam Demaree
  link: https://twitter.com/samdemaree
---
// This function retrieves the number of upvotes a Discord member has received in the past 24 hours using the Discord API.

const getDiscordUpvotes = async (memberId, apiKey, guildId, channelId, timeRangeMs) => {
    const endpoint = 'https://discord.com/api/v9'
    const timeRangeSec = Math.round(timeRangeMs / 1000)
    const time24HoursAgo = Math.round((Date.now() - timeRangeMs) / 1000)
    const headers = {
        'Authorization': `Bot ${apiKey}`,
        'Content-Type': 'application/json'
    }
    const config = {
        method: 'GET',
        headers: headers,
        url: `${endpoint}/guilds/${guildId}/audit-logs?limit=100&user_id=${memberId}&before=${time24HoursAgo}&action_type=MESSAGE_DELETE`
    }
    const response = await Functions.makeHttpRequest(config)
    if (response.error) {
        throw new Error(response.response.data.message)
    }
    const auditLogs = response.data.audit_log_entries
    let upvotes = 0
    for (let i = 0; i < auditLogs.length; i++) {
        const log = auditLogs[i]
        if (log.action_type === 72 && log.target_id === channelId && log.created_at >= time24HoursAgo - timeRangeSec) {
            upvotes++
        }
    }
    return Functions.encodeUint256(upvotes)
}
